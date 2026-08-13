# C++ Client API Reference

Full API reference for the `Tryll::` C++ client library, auto-generated from Doxygen documentation blocks in `tryll/clients/cpp/include/tryll/`. Browse individual classes and structs in the sidebar.

## Entry points

| Type                                  | Role                                                                                                                             |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `Tryll::Client::TryllClient`          | TCP session — connect, configure, manage models, create agents                                                                   |
| `Tryll::Client::ConnectedSession`     | RAII pair: owns a `ManagedServer` + `TryllClient`; returned by `RunAndConnect`                                                   |
| `Tryll::Client::AgentProxy`           | Per-agent handle — send messages, receive streaming tokens, destroy                                                              |
| `Tryll::Client::AgentVariables`       | Per-agent [Variables](https://docs.tryllengine.com/concepts/agent-variables.html) mirror — typed setters/getters, deferred flush |
| `Tryll::Client::AgentKvCacheStatus`   | Aggregate residency and reusable-prefix state for an agent                                                                       |
| `Tryll::Client::GraphDescription`     | Fluent graph builder — add nodes, wire routes, set start node                                                                    |
| `Tryll::Client::ManagedServer`        | RAII handle that spawns `tryll_server` and waits for TCP readiness                                                               |
| `Tryll::Client::ManagedServerOptions` | Configuration for `ManagedServer::Start`                                                                                         |
| `Tryll::Client::MessageResult`        | Streaming result handle returned by synchronous `SendMessage`                                                                    |
| `Tryll::Client::TryllError`           | Error type carrying a numeric code and human-readable message                                                                    |

## Supporting types

| Type                                              | Role                                                                       |
| ------------------------------------------------- | -------------------------------------------------------------------------- |
| `Tryll::Client::SessionConfig`                    | All session-configuration options, passed to `CreateSession`               |
| `Tryll::Client::GraphDescription::NodeDesc`       | Single node description inside a graph                                     |
| `Tryll::Client::GraphDescription::RouteDesc`      | Single exit-route wire inside a graph                                      |
| `Tryll::ModelInfoT`                               | Model catalog entry returned by `ListModels` (FlatBuffers object-API type) |
| `Tryll::Client::ToolDef`                          | Tool declaration passed on `CreateAgentRequest`                            |
| `Tryll::Client::ToolParamDef`                     | Single parameter within a `ToolDef`                                        |
| `Tryll::Client::AgentVariableDecl`                | Name + typed initial value, passed on `CreateAgent` to declare a variable  |
| `Tryll::Client::TryllClient::EmbeddedStorageInfo` | Embedded-storage descriptor returned by `ListEmbeddedStorages`             |

## Selected method signatures

### `TryllClient::CreateSession`

```
struct SessionConfig
{
    ::Tryll::InferenceEngine   engine          = ::Tryll::InferenceEngine_Mock;
    ::Tryll::InferenceEngine   sttEngine       = ::Tryll::InferenceEngine_Mock;
    ::Tryll::InferenceEngine   ttsEngine       = ::Tryll::InferenceEngine_Mock;
    ::Tryll::InferenceEngine   embeddingEngine = ::Tryll::InferenceEngine_Mock;
    std::string                gameName;
    std::string                storageDataFolder;   // relative storage/hotword paths resolve here
    std::chrono::milliseconds  timeout = std::chrono::seconds(30);
};

void CreateSession(const SessionConfig& cfg);
```

Each `*Engine` field selects the inference backend for that model kind independently. Engines default to `InferenceEngine_Mock` — set only the ones your session uses:

```
// Language-only session (most common)
client.CreateSession({ .engine = ::Tryll::InferenceEngine_LlamaCpp });

// Language + STT (voice input)
client.CreateSession({
    .engine    = ::Tryll::InferenceEngine_LlamaCpp,
    .sttEngine = ::Tryll::InferenceEngine_SherpaOnnx,
});
```

`CreateAgent`, `CreateEmbeddedStringStorage`, and `CreateVoiceInput` fail fast if a referenced model is not already on disk — acquire models explicitly beforehand (e.g. via `DownloadModel` or the editor Model Manager) rather than relying on the call itself to fetch them.

## `AgentProxy` callbacks

All callbacks are registered on an `AgentProxy` instance before sending the first message. They fire on the **reader thread** — they must return quickly and must not call any blocking `TryllClient` or `AgentProxy` methods.

**Dispatch priority:** typed callbacks (`SetOnToolCall`, `SetOnIntentClassified`, `SetOnPaused`) take precedence. If no typed callback is registered for an incoming `NodeEvent`, the `SetOnNodeEvent` fallback fires instead.

| Method                      | Callback type                                                                                 | Fires when                                                                                                                                                                                                                                                                                             |
| --------------------------- | --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `SetOnAnswerText(cb)`       | `void(string_view nodeName, string_view text, bool isDelta, bool isFinal)`                    | Each `AnswerText` frame during a turn. `nodeName` identifies the producing node (multi-sender graphs).                                                                                                                                                                                                 |
| `SetOnTurnComplete(cb)`     | `void(TurnStatus status, string_view debugInfoJson, int32_t tokensGenerated)`                 | `TurnComplete` arrives.                                                                                                                                                                                                                                                                                |
| `SetOnError(cb)`            | `void(const TryllError& error)`                                                               | Server-reported error or disconnect mid-turn. Does not fire for `TurnStatus_Error` turns (those arrive via `SetOnTurnComplete`).                                                                                                                                                                       |
| `SetOnToolCall(cb)`         | `void(string_view toolName, string_view argumentsJson)`                                       | `NodeEvent` with `event_type="tool_call"` (`ToolCall` node with a notifying `disposition`: `Notify`, `NotifyAndAcknowledge`, `Pause`, `PauseAndAcknowledge`, or `AwaitResult`).                                                                                                                        |
| `SetOnToolCallWithId(cb)`   | `void(string_view callId, string_view toolName, string_view argumentsJson)`                   | Same event, call-ID-aware — echo `callId` in a `ToolResult` passed to `ResumeWithToolResult(s)(Async)`.                                                                                                                                                                                                |
| `SetOnToolCallEvent(cb)`    | `void(const ToolCallEvent& event)`                                                            | Preferred form: an owning `{callId, toolName, argumentsJson, nodeName, disposition}` struct safe to capture beyond the callback (the `string_view`-based callbacks above alias the current frame).                                                                                                     |
| `SetOnIntentClassified(cb)` | `void(string_view intent, string_view recordId, size_t recordIndex, float distance)`          | `NodeEvent` with `event_type="intent_classified"` (requires `notify_client="true"` on a `ClassifyIntent` node).                                                                                                                                                                                        |
| `SetOnPaused(cb)`           | `void(string_view nodeName, string_view pendingExit)`                                         | `NodeEvent` with `event_type="paused"` — the executor paused the turn between nodes (`Pause` node, or a `ToolCall` node with `disposition` in `Pause` / `PauseAndAcknowledge` / `AwaitResult`). See [How to pause and resume a turn](https://docs.tryllengine.com/how-to/pause-and-resume-turns.html). |
| `SetOnNodeEvent(cb)`        | `void(string_view nodeName, string_view eventType, const vector<NodeEventKeyValue>& kvPairs)` | Any `NodeEvent` whose `event_type` is unrecognised or whose typed callback is not set.                                                                                                                                                                                                                 |

Pass a default-constructed `std::function` to unregister a callback.

`Resume(resumeNode = "")` / `ResumeAsync(resumeNode = "")` continue a paused turn — empty jumps via the paused node's pending exit route, non-empty jumps to that node by name. Both throw/reject with `TryllError` on `AgentNotPaused` (3012) or `UnknownNode` (3005). `ResumeWithToolResult(s)` / `ResumeWithToolResult(s)Async` attach a `ToolResult{callId, result}` batch for a paused `AwaitResult` call — required (and validated as a complete, unique batch) whenever `AwaitResult` calls are pending, or rejected with `InvalidToolResults` (3016). None of the `Resume*` overloads are reentrant-safe to call blocking from inside one of the callbacks above (they run on the reader thread); use the `*Async` overloads there.

```
// Register callbacks before the first SendText / SendMessage call.
agent.SetOnAnswerText([&](std::string_view /*nodeName*/, std::string_view text,
                          bool /*isDelta*/, bool isFinal) {
    std::cout << text;
    if (isFinal) std::cout << '\n';
});

agent.SetOnIntentClassified([&](std::string_view intent,
                                std::string_view recordId,
                                std::size_t      recordIndex,
                                float            distance) {
    std::cout << "Intent: " << intent
              << " (record " << recordId
              << ", dist " << distance << ")\n";
});

// Generic fallback for any other NodeEvent types.
agent.SetOnNodeEvent([](std::string_view nodeName,
                        std::string_view eventType,
                        const std::vector<Tryll::Client::AgentProxy::NodeEventKeyValue>& kv) {
    std::cout << "[NodeEvent] " << nodeName << " / " << eventType << '\n';
    for (auto& [k, v] : kv)
        std::cout << "  " << k << "=" << v << '\n';
});
```

See the [AgentProxy Doxygen page](https://docs.tryllengine.com/tryll-cpp/class_tryll_1_1_client_1_1_agent_proxy.html) for the full documentation of each callback type.

## `AgentProxy::Variables()`

Returns the agent's [`AgentVariables`](https://docs.tryllengine.com/concepts/agent-variables.html) mirror — typed setters (`SetInt`/`SetFloat`/`SetString`/`SetBool`/`AddToSet`/`RemoveFromSet`/`Reset`) and typed getters (`GetInt`/`GetFloat`/... returning `std::optional<T>`). Writes update the local mirror synchronously; the wire update is batched and flushed automatically immediately before the next `SendText`/`Resume`/`ChangeParams` call.

```
auto& vars = agent.Variables();
vars.SetInt("level", 13);
vars.AddToSet("quests_reached", "lost_amulet");
std::int64_t level = vars.GetInt("level").value_or(0);
```

An unknown name or type mismatch **throws** a `TryllError` immediately (no wire round-trip), using the same `3013`/`3014` codes the server would return. (The C++ setters return `void` and throw on invalid input — unlike the Unity/Unreal setters, which *return* an `FTryllError`/ `TryllError`.)

## `AgentProxy` KV-cache lifecycle

`AgentProxy` exposes `PrefillKvCacheAsync` / `PrefillKvCache`, `EvictKvCacheAsync` / `EvictKvCache`, and `GetKvCacheStatusAsync` / `GetKvCacheStatus`. The result types in `tryll/AgentKvCache.h` are aggregate: `AgentKvCacheStatus` provides `applicable`, `residency`, `prefixStatus`, `eligibleNodeCount`, and optional diagnostics; prefill adds creation/decode/reuse counts and eviction adds the evicted-context count.

Set `AgentCreateOptions::kvCacheInitialization` to the generated `AgentKvCacheInitialization` enum (`AllocateOnly`, `Prefill`, or `DeferAllocation`) when creating the agent. Calls are idle-only. Sending after eviction restores the cache automatically; there is no separate restore method. See [Manage an Agent's KV Cache](https://docs.tryllengine.com/how-to/manage-agent-kv-cache.html).

## `AgentProxy` dialog mutation

Append or tail-remove scripted user/assistant history without running the graph. Both calls are **strict idle-only** (`AgentBusy` 3004 while running, paused, or during a KV-cache lifecycle operation). Responses return counts; there is no `TurnComplete`.

```
struct DialogInteraction
{
    std::string userMessage;      // empty → omit user side
    std::string assistantMessage; // empty → omit assistant side; both empty → skipped
};

std::uint32_t appended = agent.AppendInteractions({
    {"", "Welcome! How can I help?"},           // assistant-only opener
    {"I need directions.", "Turn left at the well."},
});

std::uint32_t removed = agent.RemoveInteractionsFromEnd(1); // drop last interaction
```

Async variants: `AppendInteractionsAsync`, `RemoveInteractionsFromEndAsync`. See [Seed and edit dialog history](https://docs.tryllengine.com/how-to/seed-and-edit-dialog-history.html).

## GPU throttling and background agents

`SetInferenceThrottle` tells the server how hard to yield the GPU back to your game: `0.0f` = full speed (the default for a session that never calls it), `1.0f` = maximum yielding. It is **fire-and-forget** — no `request_id`, no response, not even an `Ack` — so it is safe to call from a render or simulation loop with nothing to await. It never changes what is generated, only how fast.

```
// From your load loop — most-stressed session wins server-wide.
client.SetInferenceThrottle(0.0f);   // full speed
client.SetInferenceThrottle(1.0f);   // hand the GPU back

// Agents nobody waits on give way first under pressure.
Tryll::AgentCreateOptions opts;
opts.workload = Tryll::AgentWorkload_Background;
auto worldLogic = client.CreateAgent(graph, opts);
```

Everything else the server classifies from the graph itself, per turn — streamed `Generate` text is protected, spoken and buffered work absorbs the slack. See [Wire Protocol](https://docs.tryllengine.com/reference/wire-protocol.html#inference-throttle-and-agent-workload) and [Server configuration](https://docs.tryllengine.com/reference/server-configuration.html#inference-scheduler).

## Headers

| Header                       | Declares                                                |
| ---------------------------- | ------------------------------------------------------- |
| `tryll/TryllClient.h`        | `TryllClient`, `ConnectedSession`, session-level types  |
| `tryll/AgentProxy.h`         | `AgentProxy` and all callback types                     |
| `tryll/AgentCreateOptions.h` | `AgentCreateOptions`, including KV-cache initialization |
| `tryll/AgentKvCache.h`       | KV-cache status and operation result types              |
| `tryll/AgentVariables.h`     | `AgentVariables`, `AgentVariableDecl`                   |
| `tryll/GraphDescription.h`   | `GraphDescription` and its nested types                 |
| `tryll/ManagedServer.h`      | `ManagedServer`, `ManagedServerOptions`                 |
| `tryll/MessageResult.h`      | `MessageResult`                                         |
| `tryll/TryllError.h`         | `TryllError`                                            |
