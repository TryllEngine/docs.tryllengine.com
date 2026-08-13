# Manage an Agent's KV Cache

Use an agent's KV-cache lifecycle controls to move language-model context memory out of a gameplay-critical moment. These controls affect per-agent LLM contexts, not shared model weights.

## Choose creation behavior

`CreateAgent` accepts `AgentKvCacheInitialization`:

| Value                    | Meaning                                                                          |
| ------------------------ | -------------------------------------------------------------------------------- |
| `AllocateOnly` (default) | Allocate eligible contexts without an explicit prefix prefill.                   |
| `Prefill`                | Allocate and prefill eligible contexts before creation completes.                |
| `DeferAllocation`        | Create context shells only; allocate when you prefill or send the first message. |

Use `Prefill` when creating an agent during a loading screen. Use `DeferAllocation` when you need an agent's graph and dialogue state now but do not need its LLM contexts until later.

## Prefill during a loading screen

Call `PrefillKvCache` after creating an idle agent to make every eligible LLM node current with its reusable prefix.

```
auto result = agent.PrefillKvCache();
if (result.applicable && result.prefixStatus == AgentKvCachePrefixStatus::Current) {
    // The agent's reusable LLM prefix is ready.
}
```

The Python equivalent is `agent.prefill_kv_cache()`. Unity and Unreal expose `PrefillKvCache` on both the raw agent and their agent components. The component APIs report completion through their KV-cache result callbacks/events; they do not provide a status UI.

## Evict an inactive agent

To release the raw contexts of an idle agent while retaining its graph, dialogue, parameters, and identity, call `EvictKvCache`.

```
auto result = agent.EvictKvCache();
// Repeating this is safe. If it was already evicted,
// result.contextsEvicted is zero.
```

Eviction is idempotent. `GetKvCacheStatus` returns the authoritative aggregate state:

- `applicable` and `eligibleNodeCount` identify whether the graph has eligible LLM nodes.
- `residency` is `Evicted` or `Resident`.
- `prefixStatus` is `NotCurrent` or `Current`.
- prefill results add `contextsCreated`, `tokensDecoded`, and `tokensReused`; eviction results add `contextsEvicted`.

## Send after eviction

There is no restore API and no option to reject a send from an evicted agent. Before every `SendMessage`, Tryll automatically recreates any evicted eligible contexts and restores their reusable prefix before it creates the interaction. The first response after eviction can therefore have a higher time to first token. Prefill explicitly when that cost belongs in a loading screen instead.

Cache operations are idle-only. A turn in progress, a paused turn, or another cache operation conflicts with `AgentBusy`. A failed explicit lifecycle operation returns `KvCacheOperationFailed` or `KvCacheManagementUnsupported`, depending on the cause.

## Use diagnostics safely

Create the agent with `enable_diagnostics` to receive optional `debug_info` in cache-operation results. It can include per-node operational detail for troubleshooting. It never includes prompt or token content, and per-node data is not part of the normal public result contract.

## What eviction does not affect

KV-cache operations apply only to `Generate`, the LLM half of `GenerateAndSpeak`, `ToolCall`, and `ClassifyIntentLLM`. They do not unload language-model weights and do not affect embeddings, `Speak`, or Sherpa-ONNX STT/TTS resources. Manage shared model weights separately through [Model Management](https://docs.tryllengine.com/reference/model-management.html).

## See also

- [Wire Protocol](https://docs.tryllengine.com/reference/wire-protocol.html)
- [Error Codes](https://docs.tryllengine.com/reference/error-codes.html)
- [Estimate Memory Footprint](https://docs.tryllengine.com/how-to/estimate-memory-footprint.html)
