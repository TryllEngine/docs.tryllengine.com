# TryllAgentComponent

**Type:** `MonoBehaviour`\
**Namespace:** `Tryll.Client`\
**Source:** `Runtime/TryllAgentComponent.cs`

Manages one [`TryllAgent`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/index.html#tryllagent) lifetime on a GameObject. Drop it on any GameObject to get an agent that connects, creates itself, and fires `UnityEvent`s as tokens arrive — no code required.

______________________________________________________________________

## Inspector fields

| Field                     | Type                          | Default        | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ------------------------- | ----------------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `WorkflowAsset`           | `TryllWorkflowAsset`          | —              | **Required.** Workflow graph asset to use. `CreateAgent` fails if this is unset. Author the graph as a [`TryllWorkflowAsset`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-workflow-asset.html) (**Create ▸ Tryll ▸ Workflow Asset**). To build a graph in code without a component, use [`TryllGraphBuilder`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/index.html#tryllgraphbuilder) and [`TryllClient.RequestCreateAgentAsync`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-client.html). |
| `EnableDiagnostics`       | `bool`                        | `false`        | When true, the server includes LLM diagnostic info in `TurnComplete` payloads.                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `MaintainDialogueHistory` | `bool`                        | `true`         | When true (default) the agent keeps its full dialogue history across turns. Set false for a stateless agent whose history is discarded after each turn — for classification/routing agents that should not store or project prior turns.                                                                                                                                                                                                                                                                                                                     |
| `KvCacheInitialization`   | `TryllKvCacheInitialization`  | `AllocateOnly` | Choose `AllocateOnly`, `Prefill`, or `DeferAllocation` for eligible language-model contexts at creation.                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `AutoCreateOnConnect`     | `bool`                        | `true`         | Automatically calls `CreateAgent()` when the session is ready (or when this component is enabled while already session-ready).                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `Speaker`                 | `TryllSpeaker`                | —              | Optional. Plays streaming TTS audio for this agent. Auto-added when the graph uses TTS and none is assigned. Leave null (and have no TTS nodes) to ignore TTS audio.                                                                                                                                                                                                                                                                                                                                                                                         |
| `VariableOverrides`       | `List<TryllVariableOverride>` | empty          | Per-instance initial-value overrides for variables declared on the workflow (Material-Instance pattern). The custom Inspector shows one optional row per declared variable — checkbox, read-only name, typed value — rather than a free-form list. Stale serialized entries (renamed/removed declarations) appear in a warning foldout and are skipped at `CreateAgent` with a `Debug.LogWarning`.                                                                                                                                                           |

### Variable Overrides Inspector

For each variable declared on the `WorkflowAsset`, the Inspector draws:

1. **Override checkbox** — when checked, this component's value replaces the declaration's initial value at `CreateAgent` (and, in Play Mode, updates the live agent mirror — see below).
1. **Name** — read-only, taken from the declaration.
1. **Value** — the type-appropriate editor (int / float / string / bool / string-set). Disabled until the override is checked; enabling seeds the field from the workflow default.

Only enabled overrides are stored as active sparse entries. Disabling an override keeps the authored value on the component (so re-enabling restores it) but clears the override flag.

**Live agent (Play Mode or Tryll Chat):** editing an override row updates `Agent.Variables` immediately (local mirror) whenever `HasAgent` is true. The wire flush stays deferred — the update is sent before the next `SendMessage` / `Resume` / `ChangeParams`. Unchecking an override assigns the **workflow declaration default** (not `Reset()`, which would restore the CreateAgent-time merged initial). A **Flush Now** button under *Live Agent — Variables* calls `FlushIfDirty` for debugging (useful before the next chat turn).

______________________________________________________________________

## Public API

```
// Runtime state
public TryllAgent Agent   { get; }
public bool       HasAgent { get; }
public TryllSpeaker Speaker { get; set; }

// Actions
public async void CreateAgent();
public void       DestroyAgent();
public new void   SendMessage(string text);
public void       Cancel(TryllCancelMode mode = TryllCancelMode.StopAndKeep);
public void       ChangeParams(string nodeName, TryllNodeParamsBase @params,
                               Action<TryllError> onComplete = null);
public void       Resume(string resumeNode = "", Action<TryllError> onComplete = null);
public void       PrefillKvCache();
public void       EvictKvCache();
public void       GetKvCacheStatus();
public void       AppendInteractions(IReadOnlyList<TryllDialogInteraction> interactions);
public void       RemoveInteractionsFromEnd(int count);
public void       ExportDialog();
public void       ImportDialog(
    TryllDialogSnapshot snapshot,
    TryllDialogCompatibility compatibility = TryllDialogCompatibility.Strict);
public TryllNodeParamsBase GetNodeParamsBaseline(string nodeName);
```

Per-agent Variables are reached through the agent handle: `agentComponent.Agent.Variables` (a [`TryllAgentVariables`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/index.html#tryllagentvariables) mirror). `Agent` is null until `OnAgentCreated` has fired.

`CreateAgent` is ignored when `HasAgent` is already true. On success it populates `Agent` and fires `OnAgentCreated`. On failure it fires `OnError` and logs to the console.

`SendMessage` fires `OnError` and logs a warning if no agent is ready.

`Cancel` cooperatively stops the in-flight turn (`OnTurnComplete` with `TryllTurnStatus.Cancelled`). No-op if no agent/turn is active.

`ChangeParams` / `Resume` complete via their result UnityEvents (`OnParamChanged` / `OnResumed`) and the optional per-call callback — exactly once per call. When no agent is ready they report `InvalidAgentId` (3001) through both channels immediately. `ChangeParams` is allowed while idle **or paused** (rejected with `AgentBusy` only while a turn is actively running between pauses).

`GetNodeParamsBaseline` returns a deep copy of the last authored params for the named node, or `null` if no agent is ready / the name is unknown. The returned object is already a clone — mutate it in place, then send via `ChangeParams`.

KV-cache operations are idle-only. Their result is aggregate status; sending after eviction restores eligible contexts automatically. This component exposes no status UI — use the result events below in your own UI if needed.

`AppendInteractions` / `RemoveInteractionsFromEnd` mutate scripted dialog history without running the graph. **Strict idle-only** — rejected with `AgentBusy` while a turn is running, paused, or during a KV-cache operation. See [Seed and edit dialog history](https://docs.tryllengine.com/v0.8/how-to/seed-and-edit-dialog-history.html).

`ExportDialog` captures the complete dialog into a `TryllDialogSnapshot`; `ImportDialog` replaces the complete dialog. Both are strict idle-only. The component reports completion through `OnDialogExported` / `OnDialogImported`. Tryll does not write a save file. See [Save and Load Agent State](https://docs.tryllengine.com/v0.8/how-to/save-and-load-agent-state.html).

______________________________________________________________________

## Events

Wire these up in the Inspector or call `AddListener` in code.

| Event                    | Signature                                                         | Description                                                                                                                                                                      |
| ------------------------ | ----------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `OnAnswerText`           | `(string nodeName, string text, bool isDelta, bool isFinal)`      | Fired for each token chunk. `nodeName` attributes the producing workflow node; `isDelta=true` when the chunk is a streaming delta; `isFinal=true` on the last chunk of the turn. |
| `OnTurnComplete`         | `(TryllTurnStatus status, string debugInfo, int tokensGenerated)` | Fired once per turn after the last `OnAnswerText`. `debugInfo` is non-empty only when `EnableDiagnostics` is true.                                                               |
| `OnError`                | `(TryllError error)`                                              | Fired on any agent-level error.                                                                                                                                                  |
| `OnAgentCreated`         | `UnityEvent`                                                      | Fired when `CreateAgent()` completes successfully.                                                                                                                               |
| `OnAgentDestroyed`       | `UnityEvent`                                                      | Fired when the agent is destroyed (by this component, by the server, or on disconnect).                                                                                          |
| `OnParamChanged`         | `(TryllError error)`                                              | Fired when a `ChangeParams` request completes. Check `error.IsOk` — `TryllError` is a value type (success is not null).                                                          |
| `OnResumed`              | `(TryllError error)`                                              | Fired when a `Resume` request completes. Check `error.IsOk`. Failures include `AgentNotPaused` (3012) and `UnknownNode` (3005).                                                  |
| `OnKvCachePrefilled`     | `(TryllKvCacheStatus result, TryllError error)`                   | Fired after a prefill request. The runtime result contains prefill counts.                                                                                                       |
| `OnKvCacheEvicted`       | `(TryllKvCacheStatus result, TryllError error)`                   | Fired after an eviction request. The runtime result contains the evicted-context count.                                                                                          |
| `OnKvCacheStatus`        | `(TryllKvCacheStatus result, TryllError error)`                   | Fired after an aggregate status request.                                                                                                                                         |
| `OnInteractionsAppended` | `(uint count, TryllError error)`                                  | Fired when `AppendInteractions` completes. `count` is `appended_count` from the server.                                                                                          |
| `OnInteractionsRemoved`  | `(uint count, TryllError error)`                                  | Fired when `RemoveInteractionsFromEnd` completes. `count` is `removed_count`.                                                                                                    |
| `OnDialogExported`       | `(TryllDialogSnapshot snapshot, TryllError error)`                | Fired when `ExportDialog` completes. Encode a successful snapshot with `ToFlatBufferBytes`.                                                                                      |
| `OnDialogImported`       | `(TryllDialogImportResult result, TryllError error)`              | Fired when `ImportDialog` completes. The result contains the restored interaction count and any compatibility warnings.                                                          |

______________________________________________________________________

## Lifetime and ownership

- The component **owns** its agent. It calls `Agent.Dispose()` in `OnDisable` and `OnDestroy`.
- Do **not** retain the `Agent` reference beyond the component's lifetime.
- If you need an agent that outlives the component, create it directly via [`TryllClient.RequestCreateAgentAsync`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-client.html) and manage its lifetime yourself.
- On disconnect the component clears its local state without sending a `DestroyAgent` wire message (the TCP connection is gone).

______________________________________________________________________

## See also

- [`TryllClient`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-client.html) — singleton session manager
- [`TryllWorkflowAsset`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-workflow-asset.html) — asset that stores a graph
- [`TryllAgent`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/index.html#tryllagent) — the raw agent handle
- [`TryllAgentVariables`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/index.html#tryllagentvariables) — the typed variables mirror
- [Agent Variables](https://docs.tryllengine.com/v0.8/concepts/agent-variables.html)
- [Change Agent Parameters at Runtime](https://docs.tryllengine.com/v0.8/how-to/change-agent-parameters.html)
- [Pause and Resume a Turn](https://docs.tryllengine.com/v0.8/how-to/pause-and-resume-turns.html)
- [Manage an Agent's KV Cache](https://docs.tryllengine.com/v0.8/how-to/manage-agent-kv-cache.html)
- [Save and Load Agent State](https://docs.tryllengine.com/v0.8/how-to/save-and-load-agent-state.html)
- [Build a chat agent with a graph](https://docs.tryllengine.com/v0.8/how-to/build-chat-agent-with-graph.html)
- [Stream answers to your UI](https://docs.tryllengine.com/v0.8/how-to/stream-answers-to-ui.html)
