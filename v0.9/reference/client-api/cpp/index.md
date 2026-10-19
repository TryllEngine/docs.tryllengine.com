# C++ Client API Reference

Full API reference for the `Tryll::` C++ client library, auto-generated from Doxygen documentation blocks in `tryll/clients/cpp/include/tryll/`. Browse individual classes and structs in the sidebar.

## Entry points

| Type                                  | Role                                                                                                                                  |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| `Tryll::Client::TryllClient`          | TCP session — connect, configure, manage models, create agents, enumerate/save/load mutable KBs                                       |
| `Tryll::Client::ConnectedSession`     | RAII pair: owns a `ManagedServer` + `TryllClient`; returned by `RunAndConnect`                                                        |
| `Tryll::Client::AgentProxy`           | Per-agent handle — `SendText`, receive streaming tokens, destroy                                                                      |
| `Tryll::Client::MutableKb`            | Experimental session-scoped named store — write/query/list/export/import                                                              |
| `Tryll::Client::AgentVariables`       | Per-agent [Variables](https://docs.tryllengine.com/v0.9/concepts/agent-variables.html) mirror — typed setters/getters, deferred flush |
| `Tryll::Client::AgentKvCacheStatus`   | Aggregate residency and reusable-prefix state for an agent                                                                            |
| `Tryll::Client::GraphDescription`     | Fluent graph builder — add typed nodes, set start node and default model                                                              |
| `Tryll::Client::ManagedServer`        | RAII handle that spawns `tryll_server` and waits for TCP readiness                                                                    |
| `Tryll::Client::ManagedServerOptions` | Configuration for `ManagedServer::Start`                                                                                              |
| `Tryll::Client::MessageResult`        | Empty placeholder reserved for future per-turn metadata. `AgentProxy::SendText` returns `void`; results arrive through the callbacks  |
| `Tryll::Client::TryllError`           | Error type carrying a numeric code and human-readable message                                                                         |

## Supporting types

| Type                                              | Role                                                                                                                                                                                                 |
| ------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Tryll::Client::SessionConfig`                    | All session-configuration options, passed to `CreateSession`                                                                                                                                         |
| `Tryll::Client::GraphDescription::NodeDesc`       | Single node inside a graph: `name` plus typed `params` (`NodeParamsVariant`)                                                                                                                         |
| `Tryll::NodeParams::<Node>ParamsT`                | Typed node params (FlatBuffers object-API types, e.g. `GenerateParamsT`, `ToolCallParamsT`) passed to `AddXxx`. Exits are string fields on these params (`<exit>_exit`); an empty exit routes to END |
| `Tryll::ModelInfoT`                               | Model catalog entry returned by `ListModels` (FlatBuffers object-API type)                                                                                                                           |
| `Tryll::MemorySnapshotT`                          | Server-wide RAM/VRAM snapshot returned by `GetMemoryConsumption`                                                                                                                                     |
| `Tryll::NodeParams::ToolDefinitionT`              | Tool declaration, stored in `ToolCallParamsT::tools` on a `ToolCall` node                                                                                                                            |
| `Tryll::NodeParams::ToolParamDefinitionT`         | Single parameter within a `ToolDefinitionT` (`name`, `type`, `description`)                                                                                                                          |
| `Tryll::Client::AgentVariableDecl`                | Name + typed initial value, passed on `CreateAgent` to declare a variable                                                                                                                            |
| `Tryll::Client::TryllClient::EmbeddedStorageInfo` | Embedded-storage descriptor returned by `ListEmbeddedStorages`                                                                                                                                       |
| `Tryll::Client::MutableKbSnapshot`                | Text-only TMKB store snapshot (vectors are never persisted)                                                                                                                                          |
| `Tryll::Client::GameMemorySnapshot`               | Whole-session mutable-KB snapshot (`save_id` + stores)                                                                                                                                               |
| `Tryll::Client::MutableKbEmbedTiming`             | `Now` or `Lazy` — required on `LoadGameMemory`; `Unspecified` is rejected                                                                                                                            |

## Selected method signatures

### `TryllClient::CreateSession`

```
struct EngineSelection
{
    ::Tryll::ModelRole       role;
    ::Tryll::InferenceEngine engine;
};

struct SessionConfig
{
    std::vector<EngineSelection>    engines;         // optional warm-up hint
    std::vector<ModelPlacementHint> placementHints;  // optional, per model
    std::string                     gameName;
    std::string                     locale;          // BCP-47, empty = unset
    std::chrono::milliseconds       timeout = std::chrono::seconds{30};
};

void CreateSession(const SessionConfig& cfg);
```

The process storage root is set when the server starts (`--storage-root` / `storage_root` / mode default). Protocol v24 removed `CreateSessionRequest.storage_data_folder`. Managed launchers pass `--project-name`, `--project-dir`, and `--storage-root` when they spawn the server.

There is no engine to choose: each model's catalog entry names the engine that runs it. `engines` is an optional hint — the server starts the named engines at `CreateSession` instead of at first use, which helps when a cold start is slow. An empty list is legal and normal. Only an out-of-range ordinal is rejected (`InvalidEngineSet`, 2005).

```
// Most sessions
client.CreateSession({});

// Warm up llama.cpp and the STT engine at session creation
client.CreateSession({
    .engines = {
        {::Tryll::ModelRole_Language, ::Tryll::InferenceEngine_LlamaCpp},
        {::Tryll::ModelRole_Stt,      ::Tryll::InferenceEngine_SherpaOnnx},
    },
});
```

`CreateAgent`, `CreateEmbeddedStringStorage`, and `CreateVoiceInput` fail fast if a referenced model is not already on disk — acquire models explicitly beforehand (e.g. via `DownloadModel` or the editor Model Manager) rather than relying on the call itself to fetch them.

### `GraphDescription` wiring and tools

There is no separate route list. Each node's typed params carry its exits as string fields (`<exit>_exit`): an empty string routes to END, and a non-empty name must match a node in the graph. Tools are `ToolDefinitionT` objects on the `ToolCall` node's `tools` field.

```
using namespace Tryll::NodeParams;

auto city = std::make_unique<ToolParamDefinitionT>();
city->name        = "city";
city->type        = "string";
city->description = "City to look up";

auto weather = std::make_unique<ToolDefinitionT>();
weather->name        = "get_weather";
weather->description = "Look up the current weather";
weather->parameters.push_back(std::move(city));

ToolCallParamsT tools;
tools.model_name          = "qwen2.5-0.5b-instruct";
tools.tools.push_back(std::move(weather));
tools.no_tool_called_exit = "chat";   // tool_called_exit left empty = END

GenerateParamsT chat;
chat.model_name = "qwen2.5-0.5b-instruct";

auto graph = Tryll::Client::GraphDescription{}
    .AddToolCall("tools", std::move(tools))
    .AddGenerate("chat", std::move(chat))
    .SetStartNode("tools");
```

### `TryllClient::GetMemoryConsumption`

```
[[nodiscard]] ::Tryll::MemorySnapshotT GetMemoryConsumption(
    std::chrono::milliseconds timeout = std::chrono::seconds{10});
```

Session-scoped (must follow `CreateSession`) but the snapshot is **server-wide**. Never throws `AgentBusy`. See [Inspect Server Memory](https://docs.tryllengine.com/v0.9/how-to/inspect-server-memory.html).

## `AgentProxy` callbacks

All callbacks are registered on an `AgentProxy` instance before the first `SendText` call. `SendText` is fire-and-forget: it returns `void`, and the turn's output arrives only through these callbacks. They fire on the **reader thread** — they must return quickly and must not call any blocking `TryllClient` or `AgentProxy` methods.

**Dispatch priority:** typed callbacks (`SetOnToolCall`, `SetOnIntentClassified`, `SetOnIntentLlmClassified`, `SetOnPaused`) take precedence. If no typed callback is registered for an incoming `NodeEvent`, the `SetOnNodeEvent` fallback fires instead.

| Method                         | Callback type                                                                                 | Fires when                                                                                                                                                                                                                                                                                                  |
| ------------------------------ | --------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SetOnAnswerText(cb)`          | `void(string_view nodeName, string_view text, bool isDelta, bool isFinal)`                    | Each `AnswerText` frame during a turn. `nodeName` identifies the producing node (multi-sender graphs).                                                                                                                                                                                                      |
| `SetOnTurnComplete(cb)`        | `void(TurnStatus status, string_view debugInfoJson, int32_t tokensGenerated)`                 | `TurnComplete` arrives.                                                                                                                                                                                                                                                                                     |
| `SetOnError(cb)`               | `void(const TryllError& error)`                                                               | Server-reported error or disconnect mid-turn. Does not fire for `TurnStatus_Error` turns (those arrive via `SetOnTurnComplete`).                                                                                                                                                                            |
| `SetOnToolCall(cb)`            | `void(string_view toolName, string_view argumentsJson)`                                       | `NodeEvent` with `event_type="tool_call"` (`ToolCall` node with a notifying `disposition`: `Notify`, `NotifyAndAcknowledge`, `Pause`, `PauseAndAcknowledge`, or `AwaitResult`).                                                                                                                             |
| `SetOnToolCallWithId(cb)`      | `void(string_view callId, string_view toolName, string_view argumentsJson)`                   | Same event, call-ID-aware — echo `callId` in a `ToolResult` passed to `ResumeWithToolResult(s)(Async)`.                                                                                                                                                                                                     |
| `SetOnToolCallEvent(cb)`       | `void(const ToolCallEvent& event)`                                                            | Preferred form: an owning `{callId, toolName, argumentsJson, nodeName, disposition}` struct safe to capture beyond the callback (the `string_view`-based callbacks above alias the current frame).                                                                                                          |
| `SetOnIntentClassified(cb)`    | `void(const IntentClassifiedEvent&)`                                                          | `NodeEvent` with `event_type="intent_classified"` (`ClassifyIntent`, `notify_client` `OnFound`/`Always`).                                                                                                                                                                                                   |
| `SetOnIntentLlmClassified(cb)` | `void(const IntentLlmClassifiedEvent&)`                                                       | `NodeEvent` with `event_type="intent_llm_classified"` (`ClassifyIntentLLM`).                                                                                                                                                                                                                                |
| `SetOnPaused(cb)`              | `void(string_view nodeName, string_view pendingExit)`                                         | `NodeEvent` with `event_type="paused"` — the executor paused the turn between nodes (`Pause` node, or a `ToolCall` node with `disposition` in `Pause` / `PauseAndAcknowledge` / `AwaitResult`). See [How to pause and resume a turn](https://docs.tryllengine.com/v0.9/how-to/pause-and-resume-turns.html). |
| `SetOnNodeEvent(cb)`           | `void(string_view nodeName, string_view eventType, const vector<NodeEventKeyValue>& kvPairs)` | Any `NodeEvent` whose `event_type` is unrecognised or whose typed callback is not set.                                                                                                                                                                                                                      |

Pass a default-constructed `std::function` to unregister a callback.

`Resume(resumeNode = "")` / `ResumeAsync(resumeNode = "")` continue a paused turn — empty follows the paused node's pending exit, non-empty jumps to that node by name. Both throw/reject with `TryllError` on `AgentNotPaused` (3012) or `UnknownNode` (3005). `ResumeWithToolResult(s)` / `ResumeWithToolResult(s)Async` attach a `ToolResult{callId, result}` batch for a paused `AwaitResult` call — required (and validated as a complete, unique batch) whenever `AwaitResult` calls are pending, or rejected with `InvalidToolResults` (3016). None of the `Resume*` overloads are reentrant-safe to call blocking from inside one of the callbacks above (they run on the reader thread); use the `*Async` overloads there.

```
// Register callbacks before the first SendText call.
agent.SetOnAnswerText([&](std::string_view /*nodeName*/, std::string_view text,
                          bool /*isDelta*/, bool isFinal) {
    std::cout << text;
    if (isFinal) std::cout << '\n';
});

agent.SetOnIntentClassified([&](const AgentProxy::IntentClassifiedEvent& ev) {
    std::cout << "Intent: " << ev.intent
              << " (record " << ev.recordId
              << ", dist " << ev.distance << ")\n";
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

See the [AgentProxy Doxygen page](https://docs.tryllengine.com/v0.9/tryll-cpp/class_tryll_1_1_client_1_1_agent_proxy.html) for the full documentation of each callback type.

## `AgentProxy::Variables()`

Returns the agent's [`AgentVariables`](https://docs.tryllengine.com/v0.9/concepts/agent-variables.html) mirror — typed setters (`SetInt`/`SetFloat`/`SetString`/`SetBool`/`AddToSet`/`RemoveFromSet`/`Reset`) and typed getters (`GetInt`/`GetFloat`/... returning `std::optional<T>`). Writes update the local mirror synchronously; the wire update is batched and flushed automatically immediately before the next `SendText`/`Resume`/`ChangeParams` call.

```
auto& vars = agent.Variables();
vars.SetInt("level", 13);
vars.AddToSet("quests_reached", "lost_amulet");
std::int64_t level = vars.GetInt("level").value_or(0);
```

An unknown name or type mismatch **throws** a `TryllError` immediately (no wire round-trip), using the same `3013`/`3014` codes the server would return. (The C++ setters return `void` and throw on invalid input — unlike the Unity/Unreal setters, which *return* an `FTryllError`/ `TryllError`.)

## `AgentProxy` KV-cache lifecycle

`AgentProxy` exposes `PrefillKvCacheAsync` / `PrefillKvCache`, `EvictKvCacheAsync` / `EvictKvCache`, and `GetKvCacheStatusAsync` / `GetKvCacheStatus`. The result types in `tryll/AgentKvCache.h` are aggregate: `AgentKvCacheStatus` provides `applicable`, `residency`, `prefixStatus`, `eligibleNodeCount`, and optional diagnostics; prefill adds creation/decode/reuse counts and eviction adds the evicted-context count.

Set `AgentCreateOptions::kvCacheInitialization` to the generated `AgentKvCacheInitialization` enum (`AllocateOnly`, `Prefill`, or `DeferAllocation`) when creating the agent. Calls are idle-only. Sending after eviction restores the cache automatically; there is no separate restore method. See [Manage an Agent's KV Cache](https://docs.tryllengine.com/v0.9/how-to/manage-agent-kv-cache.html).

## `AgentProxy` voice

An agent's voice is a generated `Tryll::VoiceProfileT` (from `tryll/schema/voice.fbs`). Pass it in `AgentCreateOptions::voice` when the graph has a `Speak` or `GenerateAndSpeak` node; without it, creation throws `TryllError` with `AgentVoiceRequired` (3025).

```
Tryll::VoiceProfileT voice;
voice.tts_model_name = "Supertonic 3 (int8)";
voice.speaker_id     = 5;

Tryll::Client::AgentCreateOptions opts;
opts.voice = std::move(voice);
auto agent = client.CreateAgent(graph, std::move(opts));

agent.SetDeliveryMode("angry", 0.8f);   // any time; from the next sentence
agent.SetVoice(otherVoice);             // idle only
```

`SetVoiceAsync` / `SetVoice` replace the voice (`AgentBusy` 3004 while a turn runs or is paused; `InvalidVoiceProfile` 3023 leaves the old voice in place). `SetDeliveryModeAsync` / `SetDeliveryMode` switch the delivery mode (`UnknownDeliveryMode` 3024). See [Voice Profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html).

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

Async variants: `AppendInteractionsAsync`, `RemoveInteractionsFromEndAsync`. See [Seed and edit dialog history](https://docs.tryllengine.com/v0.9/how-to/seed-and-edit-dialog-history.html).

`ExportDialog` / `ImportDialog` capture or replace the completed dialog as a `DialogSnapshot`. `EncodeFlatBuffer` / `DecodeFlatBuffer` produce canonical `TDLG` bytes; the game owns storage. File variants `SaveDialog` / `LoadDialog` resolve relative paths against the server's process storage root and carry a caller `save_id`. See [Export and import dialog](https://docs.tryllengine.com/v0.9/how-to/export-and-import-dialog.html) and [Save and load agent state](https://docs.tryllengine.com/v0.9/how-to/save-and-load-agent-state.html) for multi-agent envelopes, variables, and restore ordering.

## Experimental mutable knowledge bases

`TryllClient::EnsureMutableKb(name, schema)` returns a `MutableKb` handle. Enumerate, `SaveGameMemory`, and `LoadGameMemory` live on `TryllClient`, not `AgentProxy`. `LoadGameMemory` requires `MutableKbEmbedTiming::Now` or `Lazy` — `Unspecified` is rejected client-side (`8107`). Snapshots are text-only (`TMKB`); first use and eager loads re-embed. Place `Retrieve` before `WriteToMemory` in the same turn if the model should see prior memory.

```
auto store = client.EnsureMutableKb("notes", "player_log");
store.Write({Tryll::Client::MutableKbUpsert{"g1", "garlic grows in Honeywood"}});
auto hits = store.Query("where does garlic grow");
client.SaveGameMemory("saves/slot-a.tmkb", "slot-a");
client.LoadGameMemory(
    "saves/slot-a.tmkb",
    Tryll::Client::MutableKbEmbedTiming::Now);
```

## GPU throttling and background agents

`SetInferenceThrottle` tells the server how hard to yield the GPU back to your game: `0.0f` = full speed (the default for a session that never calls it), `1.0f` = maximum yielding. It is **fire-and-forget** — no `request_id`, no response, not even an `Ack` — so it is safe to call from a render or simulation loop with nothing to await. It never changes what is generated, only how fast.

```
// From your load loop — most-stressed session wins server-wide.
client.SetInferenceThrottle(0.0f);   // full speed
client.SetInferenceThrottle(1.0f);   // hand the GPU back

// Agents nobody waits on give way first under pressure.
Tryll::Client::AgentCreateOptions opts;
opts.workload = Tryll::AgentWorkload_Background;
auto worldLogic = client.CreateAgent(graph, opts);
```

Everything else the server classifies from the graph itself, per turn — streamed `Generate` text is protected, spoken and buffered work absorbs the slack. See [Wire Protocol](https://docs.tryllengine.com/v0.9/reference/wire-protocol.html#inference-throttle-and-agent-workload) and [Server configuration](https://docs.tryllengine.com/v0.9/reference/server-configuration.html#inference-scheduler).

## Headers

| Header                       | Declares                                                                        |
| ---------------------------- | ------------------------------------------------------------------------------- |
| `tryll/TryllClient.h`        | `TryllClient`, `ConnectedSession`, session-level types                          |
| `tryll/AgentProxy.h`         | `AgentProxy` and all callback types                                             |
| `tryll/AgentCreateOptions.h` | `AgentCreateOptions`, including KV-cache initialization and the agent's `voice` |
| `tryll/AgentKvCache.h`       | KV-cache status and operation result types                                      |
| `tryll/AgentVariables.h`     | `AgentVariables`, `AgentVariableDecl`                                           |
| `tryll/GraphDescription.h`   | `GraphDescription` and its nested types                                         |
| `tryll/ManagedServer.h`      | `ManagedServer`, `ManagedServerOptions`                                         |
| `tryll/MessageResult.h`      | `MessageResult`                                                                 |
| `tryll/TryllError.h`         | `TryllError`                                                                    |
| `tryll/MutableKb.h`          | `MutableKb` handle and per-op result types                                      |
| `tryll/MkbSnapshot.h`        | TMKB / GameMemory snapshot DTOs and codecs                                      |
