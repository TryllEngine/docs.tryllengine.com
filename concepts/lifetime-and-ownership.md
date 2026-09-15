# Lifetime and Ownership

Three kinds of resources in Tryll outlive any single wire request: [string storages](https://docs.tryllengine.com/reference/string-storage.html), [embedded string storages](https://docs.tryllengine.com/reference/embedded-string-storage.html), and [language / embedding models](https://docs.tryllengine.com/reference/model-management.html). Agents reference them by name, often for the whole conversation. This page explains *who owns each one*, *how long it stays alive*, and *what actually happens when you destroy it* — so you can reason about the edge cases without reading the server source.

## The dual-ownership pattern

Every resource of the three kinds follows the same shape:

- A **manager** owns a shared reference plus a **retention mode** (`Pinned` or `OnDemand`). The manager is the surface you see on the wire — it answers `Create…` / `Destroy…` requests.
- Each **node** that needs the resource is given its own shared reference at [`CreateAgent`](https://docs.tryllengine.com/reference/agent-parameters.html) time.
- The resource lives as long as **any** reference still exists — manager or node, whichever is last.

Consequence: destroying a storage from the client never yanks data out from under a live agent. `Destroy…Request` drops only the manager's reference (or demotes it to OnDemand). Any agent whose nodes still hold the resource finishes cleanly; the bytes are freed once the *last* reference drops.

The only difference between the three resource kinds is *where* the manager lives:

| Resource              | Manager scope                        | Lives until                                                                                                             |
| --------------------- | ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------- |
| StringStorage         | **Session**                          | Session closes, or `DestroyStringStorageRequest` (Unpin / unregister) + the last agent holding it is destroyed.         |
| EmbeddedStringStorage | **Session**                          | Session closes, or `DestroyEmbeddedStringStorageRequest` (Unpin / unregister) + the last agent holding it is destroyed. |
| Model                 | **Process** (shared across sessions) | The server unloads it (`UnloadModelRequest` for Pinned, or automatically for OnDemand once unused).                     |

The rest of this page walks through each case.

## StringStorage

The preferred way to supply a string storage to a node is to set the node's `string_storage` param to a **relative file path** under the session's `storage_data_folder`. The server loads the file on demand at `CreateAgent` time — no explicit `Create…` / `Destroy…` wire round-trip is needed.

When you need to share the same inline data across multiple agents, use an explicit [`CreateStringStorageRequest`](https://docs.tryllengine.com/reference/string-storage.html), which Pins the storage in the session manager.

```
flowchart LR
    subgraph Session
        Mgr["StringStorageManager\n(Pinned | OnDemand)"]
    end
    subgraph Agent
        N1[CannedResponse node]
        N2[RegexGuardrail node]
    end
    Mgr -- "shared ref" --> S[(StringStorage)]
    N1  -- "shared ref" --> S
    N2  -- "shared ref" --> S
```

**Retention modes:**

- **Pinned** — set by `CreateStringStorageRequest` (explicit Pin) or when `NodeFactory` loads a file-backed storage at `CreateAgent` time. The manager holds a reference of its own; `EvictUnusedOnDemand()` will not evict it.
- **OnDemand** — the default for lazy file-backed loads that are then Unpinned (e.g. after `DestroyStringStorageRequest`). The manager releases its reference; once the last node drops its reference the storage is freed.

**What each operation actually does:**

- `CreateStringStorageRequest` — loads or registers the storage and Pins it. Server responds with `CreateStringStorageResponse`.
- `DestroyStringStorageRequest` — for file-backed storages: demotes from Pinned to OnDemand (eligible for eviction once no node holds it). For virtual / inline storages: removes the named entry. The node references are unaffected in either case.
- `CreateAgentRequest` (with a relative-path `string_storage` param) — the session's `NodeFactory` resolves the path, loads-or-gets the storage from the manager, Pins it while the agent is alive, and gives the node a shared reference.
- `DestroyAgentRequest` — each node's shared reference drops. If the storage was loaded as part of this agent and has since been Unpinned (or was never explicitly Pinned), the manager demotes to OnDemand and the entry is evicted on the next `EvictUnusedOnDemand` call.
- Session close — the manager is torn down, then every remaining agent is destroyed. The last reference goes with the last agent.

Updating a storage in place is not supported — create a new one under a different name (and, if needed, rebind via [`change_param`](https://docs.tryllengine.com/how-to/change-agent-parameters.html)) instead of mutating one that agents already hold.

## EmbeddedStringStorage

The preferred way to supply a knowledge base is to set the node's `embedded_string_storage` param to a **relative path** pointing at the KB config JSON under the session's `storage_data_folder`. The server loads the config and builds (or reuses a cached) HNSW index at `CreateAgent` time — no explicit `Create…` is needed.

```
flowchart LR
    subgraph Session
        Mgr["EmbeddedStringStorageManager\n(Pinned | OnDemand)"]
    end
    subgraph Agent
        R[Retrieve node]
    end
    Mgr -- "shared ref" --> E[(EmbeddedStringStorage<br>records + HNSW index)]
    R   -- "shared ref" --> E
```

The in-memory object — records, embeddings, HNSW index — follows the same dual-ownership rules as `StringStorage`, including Pinned vs OnDemand retention.

**Create = Pin; Destroy = Unpin:**

- `CreateEmbeddedStringStorageRequest (config_path)` — resolves the relative path against the session root, loads via `FindOrLoad` (reusing a cached entry if available), and Pins the entry.
- `CreateEmbeddedStringStorageRequest (strings)` — builds an in-memory index and registers it as a **virtual** entry under `name`. Always Pinned while registered; evicted when destroyed.
- `DestroyEmbeddedStringStorageRequest` — for file-backed (Path A) entries: demotes from Pinned to OnDemand. For virtual (Path B) entries: removes the entry. Node references are unaffected.

**On-disk artifacts are separate.** A Path-A storage reads from a records file and optionally a pre-built `.usearch` index on the server's disk. Those files are *not* owned by the session; they remain on disk after the session ends and are reused on the next request that points at the same config. What is rebuilt per session is the in-memory index object, not the cached bytes on disk.

## Models

Models follow the same manager + node pattern, but the manager lives on the **server process**, not on a single session. That is why pinning a model in one session keeps it loaded for every other session on the same server.

```
flowchart LR
    subgraph Server process
        MM[ModelManager]
    end
    subgraph Session A
        GA[Generate / ToolCall node]
    end
    subgraph Session B
        GB[Generate / ToolCall node]
    end
    MM -- "shared ref<br>(only while Pinned)" --> M[(Model)]
    GA -- "shared ref" --> M
    GB -- "shared ref" --> M
```

Two things decide how long the model stays resident. The same rules apply to **language, embedding, STT, and TTS** (each kind has its own cache; VAD is catalog/download-only and follows the STT model that loads it):

- **Retention mode.** `LoadModelRequest` installs the model as **Pinned** — the `ModelManager` holds a reference of its own. A model loaded implicitly (because an agent's graph, voice input, TTS, or embedding use referenced it without a prior `LoadModelRequest`) is **OnDemand** — only the live users hold references beyond the cache entry.
- **Active users.** Each `Generate` / `ToolCall` context (and STT / TTS / embedding holder) keeps a reference for the life of its agent or voice-input session.

The model is unloaded when the last reference goes away *and* the manager decides to drop the cache entry:

- **Pinned** → released by `UnloadModelRequest`, but only after every user has been torn down. If agents are still active the request is acknowledged immediately and the actual unload is deferred until the last context drops.
- **OnDemand** → the server runs `EvictUnusedOnDemand` after every `DestroyAgentRequest`, and again immediately before the next validated load into any of the four caches (cross-cache sweep). No explicit unload call is needed.

## Session end

When the TCP connection closes (or the server shuts down), a single cleanup sequence runs for the session:

1. Every active turn is cancelled.
1. Every agent in the session is destroyed.
1. The session's `StringStorageManager` and `EmbeddedStringStorageManager` are torn down, dropping their references.
1. After the last agent is gone, `EvictUnusedOnDemand` frees any OnDemand models whose last user just left. Any idle OnDemand model that briefly remains cached is swept when the next validated model load starts.

Pinned models survive — that is the whole point of pinning. A fresh session can reuse them without paying the load cost again.

## Common questions

**Do I need to call `CreateStringStorageRequest` before `CreateAgentRequest`?** No. Set the node's `string_storage` to a relative file path and the server loads it automatically at `CreateAgentRequest` time. Explicit `Create…` is only needed for shared inline data or to pre-warm a Pinned entry.

**Can I `destroy_string_storage` while an agent is running?** Yes. The call is safe at any time. Live nodes keep the data alive for the rest of the agent's life; the storage name just becomes available for reuse in the same session.

**Can I use any name for an inline storage?** The name must not resolve to an existing real file under `storage_data_folder`. If it does, the registration is rejected with `7004 StorageNameInUse` to prevent the virtual entry from silently shadowing a file with the same name.

**Do I need to re-pin my model after each agent?** No. Pinned stays loaded across agent create / destroy cycles — and even across sessions on the same server process. Only `UnloadModelRequest` demotes it.

**Is the Path-A `.usearch` index rebuilt every session?** No. The cached index on disk is reused; only the in-memory index object follows the normal shared-reference rules.

**What if two agents reference the same storage?** Each holds its own reference. Destroying one agent drops that agent's reference; the other keeps using the storage normally.

**Can I mutate a string storage in place?** No — there is no wire request to edit one. Create a new storage and, if a live agent needs the new content, use [`change_param`](https://docs.tryllengine.com/how-to/change-agent-parameters.html) to rebind the node's `string_storage` param.

## Related

- [Reference: String Storage](https://docs.tryllengine.com/reference/string-storage.html)
- [Reference: Embedded String Storage](https://docs.tryllengine.com/reference/embedded-string-storage.html)
- [Reference: Model Management](https://docs.tryllengine.com/reference/model-management.html)
- [Concept: Agents and Sessions](https://docs.tryllengine.com/concepts/agents-and-sessions.html)
- [Concept: Models and Inference Engines](https://docs.tryllengine.com/concepts/models-and-inference-engines.html)
- [How-to: Pin and unpin models](https://docs.tryllengine.com/how-to/pin-and-unpin-models.html)
- [How-to: Change agent parameters at runtime](https://docs.tryllengine.com/how-to/change-agent-parameters.html)
