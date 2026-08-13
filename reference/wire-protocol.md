# Wire Protocol

This page specifies the byte-level contract a third-party client must implement to talk to a Tryll server without using the shipped C++, Python, or Unreal client libraries. If you are using one of those libraries, you do not need any of this — skip to the [how-to guides](https://docs.tryllengine.com/how-to/index.html).

The authoritative source of truth for every structure named below is the FlatBuffers schema file `tryll/schema/messages.fbs`. The current **protocol version is 10** (sent in `ConnectionReady.protocol_version`). This reference describes the transport around that schema.

## Transport

- **TCP** only. No TLS; the server is expected to run on `localhost` or inside a trusted network.
- **Bi-directional**. Both client-to-server requests and server-to-client responses use the same framing over the same socket.
- **One connection = one [session](https://docs.tryllengine.com/reference/glossary.html#session).** Closing the socket destroys every [agent](https://docs.tryllengine.com/reference/glossary.html#agent) owned by the session.

## Framing

Every frame is a **length-prefixed FlatBuffers payload**:

```
+-------------------+----------------------------------+
|  length (4 bytes) |  FlatBuffers root table (N bytes)|
|  little-endian    |                                  |
|  uint32           |                                  |
+-------------------+----------------------------------+
```

- `length` is the byte count of the FlatBuffers payload that follows, little-endian.
- The payload's root type is `Message` (see `messages.fbs`); `Message.body` is a union whose variant tag selects the message kind.
- **Maximum frame size: 1 MiB (1 048 576 bytes).** Any frame exceeding the cap is rejected with error `5003` `FrameTooLarge` (see [error codes](https://docs.tryllengine.com/reference/error-codes.html#5xxx-protocol-framing)).

No compression, no chunking below the FlatBuffers level, no keep-alive heartbeats. The client is expected to read frames in a loop and dispatch them by the `body` union tag.

## Message types

All messages are defined in `messages.fbs` under the `MessageBody` union. The full catalog (table names as they appear in the schema):

| Direction | Request                                | Corresponding response(s)                                                                             |
| --------- | -------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| S → C     | —                                      | `ConnectionReady` (unsolicited, sent on accept)                                                       |
| C → S     | `CreateSessionRequest`                 | `CreateSessionResponse`                                                                               |
| C → S     | `CreateAgentRequest`                   | `CreateAgentResponse`                                                                                 |
| C → S     | `SendMessageRequest`                   | `AnswerText` × N, then `TurnComplete`                                                                 |
| C → S     | `PrefillAgentKvCacheRequest`           | `PrefillAgentKvCacheResponse`                                                                         |
| C → S     | `EvictAgentKvCacheRequest`             | `EvictAgentKvCacheResponse`                                                                           |
| C → S     | `GetAgentKvCacheStatusRequest`         | `GetAgentKvCacheStatusResponse`                                                                       |
| C → S     | `AppendInteractionsRequest`            | `AppendInteractionsResponse`                                                                          |
| C → S     | `RemoveInteractionsFromEndRequest`     | `RemoveInteractionsFromEndResponse`                                                                   |
| C → S     | `CancelRequest`                        | `Ack` (the in-flight turn ends with `TurnComplete`, `status = Cancelled`)                             |
| C → S     | `ResumeAgentRequest`                   | `Ack`, or `ErrorResponse` (`AgentNotPaused` / `UnknownNode`)                                          |
| C → S     | `UpdateAgentVariablesRequest`          | `Ack`, or `ErrorResponse` (`AgentBusy` / `UnknownVariable` / `VariableTypeMismatch`)                  |
| C → S     | `GetAgentVariablesRequest`             | `GetAgentVariablesResponse`                                                                           |
| C → S     | `ChangeAgentParamRequest`              | `Ack`, or `ErrorResponse` (`AgentBusy` / `UnknownNode` / `ParamNotMutable` / `InvalidParamValue`)     |
| C → S     | `DestroyAgentRequest`                  | `Ack`                                                                                                 |
| C → S     | `ListModelsRequest`                    | `ListModelsResponse`                                                                                  |
| C → S     | `DownloadModelRequest`                 | `DownloadProgress` × N, then `DownloadComplete`                                                       |
| C → S     | `LoadModelRequest`                     | `LoadModelResponse`                                                                                   |
| C → S     | `UnloadModelRequest`                   | `Ack`                                                                                                 |
| C → S     | `DeleteModelRequest`                   | `Ack` (removes downloaded files from disk)                                                            |
| C → S     | `CreateStringStorageRequest`           | `CreateStringStorageResponse`                                                                         |
| C → S     | `DestroyStringStorageRequest`          | `Ack`                                                                                                 |
| C → S     | `CreateEmbeddedStringStorageRequest`   | `CreateEmbeddedStringStorageResponse`                                                                 |
| C → S     | `DestroyEmbeddedStringStorageRequest`  | `Ack`                                                                                                 |
| C → S     | `CreateVoiceInputRequest`              | `CreateVoiceInputResponse`                                                                            |
| C → S     | `BeginUtteranceRequest`                | `Ack`                                                                                                 |
| C → S     | `AudioBuffer`                          | — (fire-and-forget PCM; correlated by `voice_input_id`, no `request_id`)                              |
| C → S     | `EndUtteranceRequest`                  | `Ack`                                                                                                 |
| C → S     | `CancelUtteranceRequest`               | `Ack`                                                                                                 |
| C → S     | `DestroyVoiceInputRequest`             | `Ack`                                                                                                 |
| C → S     | `SetInferenceThrottle`                 | — (fire-and-forget; session-scoped, no `request_id`)                                                  |
| S → C     | `WireTranscriptUpdate`                 | (unsolicited; `kind` = `SpeechStart` / `Partial` / `SegmentFinal` / `UtteranceFinal`)                 |
| S → C     | `TtsAudioFormatFrame`, `TtsAudioFrame` | (unsolicited during a turn with a `Speak` / `GenerateAndSpeak` node; end signalled by `TurnComplete`) |
| S → C     | `ErrorResponse`                        | (replaces any expected response on failure)                                                           |
| S → C     | `NodeEvent`                            | (unsolicited, fire-and-forget)                                                                        |

Development-only messages

`DebugCommandRequest` / `DebugCommandResponse` form a diagnostic channel that is **excluded from production server builds** and disabled by default otherwise. It is intended for Tryll's own testing (e.g. crashing the server on purpose to validate crash capture) and is not part of the stable integration surface — production servers reject it.

## Session lifecycle

A connection follows this sequence:

```
sequenceDiagram
    participant C as Client
    participant S as Server
    C->>S: TCP connect
    S-->>C: ConnectionReady(protocol_version, codegen_fingerprint)
    C->>S: CreateSessionRequest(inference_engine, stt_engine?, tts_engine?, embedding_engine?)
    S-->>C: CreateSessionResponse
    C->>S: CreateAgentRequest(graph, ...)
    note over S: fails fast (GraphCompilationFailed) if a referenced model is absent
    S-->>C: CreateAgentResponse(agent_id)
    loop per turn
        C->>S: SendMessageRequest(agent_id, text)
        S-->>C: AnswerText(chunk, is_final=false)
        S-->>C: AnswerText(chunk, is_final=true)
        S-->>C: TurnComplete(status, debug_info?)
    end
    C->>S: DestroyAgentRequest(agent_id)
    S-->>C: Ack
    C->>S: TCP close
```

The server emits `ConnectionReady` **before** any request has been sent; a client that issues `CreateSessionRequest` without first reading `ConnectionReady` races against the schema-version check carried in that frame.

`CreateSessionRequest` carries one engine per model kind: `inference_engine` for language models, plus `stt_engine`, `tts_engine`, and `embedding_engine`. Each defaults to `Mock`; set only the ones you actually use. The server dispatches model lookups by the catalog `ModelType` of the model name in question, so `ListModels`, `LoadModelRequest`, and `UnloadModelRequest` work uniformly across language / embedding / STT / TTS. VAD is catalog/download-only — `LoadModelRequest` for a VAD model returns `ModelResolutionFailed`.

It also carries `storage_data_folder: string` — when non-empty, this becomes the **per-session storage root** against which relative paths in node `string_storage` and `embedded_string_storage` params are resolved. Falls back to the server's `storage_root` config field, then the exe directory. See [Server Configuration](https://docs.tryllengine.com/reference/server-configuration.html#storage-root) for the full resolution chain.

`CreateVoiceInputRequest` now carries `hotwords_storage_path: string` — a relative path (resolved against the session storage root) to a plain `.txt` file listing hotwords, one per line, and `hotwords_score: float` (threshold score for keyword activation; default `1.5`). These are exposed as `HotwordsStoragePath` and `HotwordsScore` UPROPERTYs on the Unreal voice component.

Important properties:

1. **Request IDs are optional.** Every request may carry a client-assigned `request_id`; the server echoes it back on the matching response and on any `ErrorResponse` that replaces that response. Use it to correlate when multiple requests are in flight.
1. **Only one turn per agent at a time.** Sending a second `SendMessageRequest` before `TurnComplete` arrives yields error `3004` `AgentBusy`.
1. **Multiple agents per session.** A session may own many agents; turns on different agents run in parallel.
1. **Unsolicited frames.** The server may emit `NodeEvent` or other unsolicited frames at any time. Clients must tolerate frames arriving between an in-flight request and its response.
1. **`CreateAgent` does not auto-download.** Models referenced by the graph must already be on disk; the server fails fast with `GraphCompilationFailed` if one is missing, rather than downloading it and streaming `DownloadProgress` under the request. Use `DownloadModelRequest` (or the editor Model Manager) to acquire models beforehand.

## Agent KV-cache lifecycle

`CreateAgentRequest.kv_cache_initialization` controls how eligible language-model contexts are materialised:

| Value                 | Meaning                                                                   |
| --------------------- | ------------------------------------------------------------------------- |
| `AllocateOnly` (0)    | Allocate contexts without explicit prefill (default).                     |
| `Prefill` (1)         | Allocate and prefill every eligible context before `CreateAgentResponse`. |
| `DeferAllocation` (2) | Create context shells only; materialise on prefill or send.               |

`PrefillAgentKvCacheRequest`, `EvictAgentKvCacheRequest`, and `GetAgentKvCacheStatusRequest` are independent unary operations. Their dedicated responses — not `TurnComplete` — contain the final aggregate state: `applicable`, `residency` (`Evicted` / `Resident`), `prefix_status` (`NotCurrent` / `Current`), `eligible_node_count`, and optional `debug_info`. Prefill additionally reports `contexts_created`, `tokens_decoded`, and `tokens_reused`; eviction reports `contexts_evicted`.

The server restores evicted eligible contexts before every `SendMessageRequest` and before creating the interaction. There is no restore request, no reject-send mode, and no `AgentKvCacheEvicted` error. Cache operations are idle-only; conflicts with a running or paused turn, or another cache operation, return `AgentBusy`.

Per-node lifecycle detail is diagnostic-only: it can appear in `debug_info` when `CreateAgentRequest.enable_diagnostics` is true. It never contains prompt or token content.

## Dialog history mutation

Protocol v7 adds two idle-only request/response pairs for appending or tail-removing scripted dialog **without running the workflow graph**. They return dedicated responses with counts — no `AnswerText` or `TurnComplete`.

### AppendInteractions

`AppendInteractionsRequest {agent_id, request_id?, interactions: [DialogInteraction]}`

Each `DialogInteraction` is `{user_message: string, assistant_message: string}`:

| Side                | Empty string | Effect                                                |
| ------------------- | ------------ | ----------------------------------------------------- |
| `user_message`      | yes          | User slot omitted for that interaction                |
| `assistant_message` | yes          | Assistant slot omitted for that interaction           |
| both                | yes          | Pair skipped (does not count toward `appended_count`) |

Non-empty sides become `SlotComponent`s on a new `Interaction`: the user side uses the reserved `user_message` slot (`SlotKind::User`); the assistant side uses `assistant_message` with `history_role = Assistant` and producer `client_injected`. An empty `interactions` array, or a batch whose pairs are all both-empty, succeeds with `appended_count = 0`.

`AppendInteractionsResponse {request_id?, agent_id, appended_count}` reports how many interactions were actually appended.

### RemoveInteractionsFromEnd

`RemoveInteractionsFromEndRequest {agent_id, request_id?, count: uint32}` removes whole interactions from the **tail** of the dialog. `count = 0` is a no-op (`removed_count = 0`). `count >= dialog size` clears all interactions (saturating). `RemoveInteractionsFromEndResponse {request_id?, agent_id, removed_count}` reports how many were removed.

### Admission, KV, and projection

Both operations require a **strictly idle** agent: rejected with `3004 AgentBusy` while a turn is running, while a turn is **paused**, or while a KV-cache lifecycle operation is in flight. This is stricter than [`ChangeAgentParam`](https://docs.tryllengine.com/how-to/change-agent-parameters.html), which is allowed while paused.

Either mutation invalidates the agent's reusable KV prefix (`_kvCachePrefixCurrent = false`). Removal additionally resets the token-budget projection window on every language-model context. The next `SendMessage` pays re-sync / re-prefill cost for the edited suffix.

**Stateless agents** (`maintain_dialogue_history = false`) may append scripted history as a one-shot seed before a single `SendMessage`; the dialogue is cleared after that turn completes, as with any other turn.

**vs `StopAndDiscard`.** [`Cancel`](https://docs.tryllengine.com/how-to/cancel-a-turn.html) with `StopAndDiscard` removes the **last in-flight** interaction after a cooperative cancel mid-turn. Dialog mutation edits completed or injected history on an idle agent; tail removal is explicit and count-based.

The Unity (`TryllAgent.AppendInteractions` / `RemoveInteractionsFromEnd`), Unreal (`FTryllAgent::AppendInteractions` / `RemoveInteractionsFromEnd`), C++ (`AgentProxy::AppendInteractions` / `RemoveInteractionsFromEnd`), and Python (`AgentProxy.append_interactions` / `remove_interactions_from_end`) clients expose these directly. See [Seed and edit dialog history](https://docs.tryllengine.com/how-to/seed-and-edit-dialog-history.html).

## Streaming answers

For every [turn](https://docs.tryllengine.com/reference/glossary.html#turn), the server emits **one or more** `AnswerText` frames followed by exactly one `TurnComplete`:

| Field       | Meaning                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `agent_id`  | Echoes the target agent.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `node_name` | Name of the workflow node that produced this chunk (multi-sender attribution — a graph can route through more than one text-producing node in a single turn, e.g. two `CannedResponse` nodes voicing two NPCs). Clients that only care about the text may ignore this field.                                                                                                                                                                                                                                             |
| `text`      | The text chunk.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `is_delta`  | `true` for delta chunks, `false` when `text` holds the accumulated response so far.                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `is_final`  | `true` on the last chunk from a given node. Each text-producing node emits its own final chunk for its `node_name`; a turn with a single producing node still has exactly one `is_final = true` chunk overall, but a turn that routes through multiple producing nodes (or a node whose `send = None`, which never emits) will have one `is_final = true` per node that emitted. A turn that produces no text at all (e.g. routed to a canned response with empty output) still emits one final chunk with empty `text`. |

`TurnComplete` carries:

| Field              | Meaning                                                                                                                                       |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `agent_id`         | Echoes the target agent.                                                                                                                      |
| `status`           | `TurnStatus::Success`, `Error`, or `Cancelled`.                                                                                               |
| `debug_info`       | JSON string with per-node execution data; populated only when the agent was created with `enable_diagnostics = true`. Empty string otherwise. |
| `tokens_generated` | Total tokens sampled across all generation nodes in this turn (prompt tokens excluded).                                                       |

## Cancelling a turn

Send `CancelRequest` `{agent_id, mode}` to stop the agent's in-flight turn. The server replies with `Ack` and the active turn ends with `TurnComplete` (`status = Cancelled`). Generation stops cooperatively — the current step finishes, then text generation stops at the next token and speech (TTS) stops after the current audio chunk. Cancelling an idle agent is a no-op.

`mode` (`CancelMode`):

| Value            | Meaning                                                                                                                                                                              |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `StopAndKeep`    | Stop, but keep the partial reply in history. This is what a chat "Stop" button uses.                                                                                                 |
| `StopAndDiscard` | Stop and discard the whole interaction (the user turn and partial reply), as if it never happened. Useful when the agent is a time-boxed decision maker and no decision was reached. |

The Unity (`TryllAgent.Cancel` / `TryllAgentComponent.Cancel`), Unreal (`UTryllAgentComponent::Cancel` / `FTryllAgent::Cancel`), C++ (`AgentProxy::Cancel`), and Python (`AgentProxy.cancel`) clients expose this directly. The built-in editor chat windows replace **Send** with **Stop** while a turn streams.

## Pausing and resuming a turn

Two node-level mechanisms suspend a turn **between nodes**, mid-stream:

- a [`Pause`](https://docs.tryllengine.com/reference/nodes/pause.html) node — a no-op checkpoint; the executor always pauses after it,
- [`ToolCall`](https://docs.tryllengine.com/reference/nodes/tool-call.html) with a pausing `disposition` (`Pause`, `PauseAndAcknowledge`, `AwaitResult`) — pauses only when a tool call was detected (the `tool_called` exit).

While paused, the turn is still open (`SendMessage` still gets `AgentBusy`), but [`ChangeAgentParam`](https://docs.tryllengine.com/how-to/change-agent-parameters.html) — normally rejected while a turn is running — is allowed. This is the point of pausing: react to a tool call (or any checkpoint) by mutating node params before the graph continues. There is no pause timeout — `CancelRequest` is the only way to abort a stuck pause.

The server announces a pause with a `NodeEvent` (`event_type = "paused"`, `kv_pairs = {exit_route}`) — the same fire-and-forget, unsolicited channel used for `tool_call` and `intent_classified` (see [Tool Call](https://docs.tryllengine.com/reference/nodes/tool-call.html)). Protocol v8 added `ResumeAgentRequest.tool_results`; protocol v9 consolidates ToolCall notify / pause / history into `disposition`. Resumes use `ResumeAgentRequest {agent_id, resume_node, tool_results[]}`:

| `resume_node` | Effect                                                                                                                            |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Empty         | Continue via the paused node's wired exit route (the `exit_route` from the `paused` event).                                       |
| Non-empty     | Jump directly to that node by name, skipping the wired exit. `ErrorResponse(UnknownNode, 3005)` if it doesn't exist in the graph. |

Each optional `ToolResult` is `{call_id, result}`; `call_id` must match the stable id from the `tool_call` event. A non-empty batch must contain exactly one result for every pending `AwaitResult` call. The server validates the whole batch and attaches all results before waking the turn. Unknown, duplicate, missing, non-tool-pause, or oversized batches return `ErrorResponse(InvalidToolResults, 3016)` and leave the turn paused for retry. An empty batch preserves plain continue behavior.

`ResumeAgentRequest` while the agent is idle, or actively running (not paused), returns `ErrorResponse(AgentNotPaused, 3012)`.

Clients expose plain continue/jump via `Resume` / `resume`, and tool-result batches via `ResumeWithToolResult(s)` (Unity / Unreal / C++) or `resume(..., tool_results=)` (Python). Correlation ids arrive on `ToolCallNotificationWithId` / `OnToolCallWithId` / `SetOnToolCallWithId` / `set_on_tool_call_with_id` (legacy name-only callbacks still fire). Unity and Unreal also offer `RegisterTool` sugar that auto-resumes a complete paused batch when every call has a registered handler. Auto-resume attaches `tool_results` only for `await_result`; acknowledgement / route-only dispositions resume payload-free. A payload-free Resume while `AwaitResult` calls are still pending is rejected with `3016 InvalidToolResults`. Typed `paused` callbacks: `SetOnPaused` / `set_on_paused` / Unity `Paused` / Unreal `OnPaused`. See [How to pause and resume a turn](https://docs.tryllengine.com/how-to/pause-and-resume-turns.html) and [Define and handle tool calls](https://docs.tryllengine.com/how-to/define-and-handle-tool-calls.html).

## Agent Variables

`CreateAgentRequest` carries a trailing `variables: [AgentVariable]` field — the full, type-locked declaration of the agent's [Variables](https://docs.tryllengine.com/concepts/agent-variables.html) store, each entry a `{name: string, value: VariableValue}` pair. `VariableValue` is a union over `VarInt` (int64), `VarFloat` (float64), `VarString`, `VarBool`, and `VarStringSet` (`[string]`).

`UpdateAgentVariablesRequest {agent_id, request_id?, updates: [VariableUpdate]}` applies a batch of mutations atomically — every update is validated before any is applied, so a rejected batch never partially mutates the store. Each `VariableUpdate` is `{name: string, op: VariableOp, value: VariableValue?}`:

| `op`            | Requires `value`?   | Effect                                                                    |
| --------------- | ------------------- | ------------------------------------------------------------------------- |
| `Assign`        | yes                 | Replace the variable's current value.                                     |
| `AddToSet`      | yes, `VarString`    | Insert a string into a `set<string>` variable (no-op if already present). |
| `RemoveFromSet` | yes, `VarString`    | Remove a string from a `set<string>` variable (no-op if already absent).  |
| `Reset`         | no (must be absent) | Restore the `CreateAgent`-time initial value.                             |

The server replies `Ack` on success, or `ErrorResponse` on the first invalid update in the batch (`3013 UnknownVariable`, `3014 VariableTypeMismatch`, or `3004 AgentBusy` — admission mirrors [`ChangeAgentParam`](https://docs.tryllengine.com/how-to/change-agent-parameters.html): allowed while idle or paused, rejected while a turn is actively running).

`GetAgentVariablesRequest {agent_id, request_id?}` returns `GetAgentVariablesResponse {request_id?, variables: [AgentVariable]}` — a full snapshot of the agent's current values. This response carries no `agent_id`; correlate multiple in-flight `Get` calls via `request_id`.

Declared variables are consumed server-side in two places: Mustache templates via `{{var.<name>}}` (see [Use Mustache templates](https://docs.tryllengine.com/how-to/use-mustache-templates.html)) and the Retrieve/ClassifyIntent filter grammar's `{"var": "<name>"}` operand (see [Retrieve filter grammar](https://docs.tryllengine.com/reference/nodes/retrieve-filter-grammar.html)) — both re-read the current value on every use, so a mutation takes effect on the very next turn with no recompile.

## Voice input and transcripts

When a VoiceInput utterance is active the server streams `WireTranscriptUpdate` frames to the client. Each frame carries a `kind` field that classifies the event:

| `kind`           | Value | Meaning                                                                                                                                                                                                                                                |
| ---------------- | ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `SpeechStart`    | 0     | VAD rising edge; `text` is empty.                                                                                                                                                                                                                      |
| `Partial`        | 1     | Revisable in-progress hypothesis. Overwrite any previous partial.                                                                                                                                                                                      |
| `SegmentFinal`   | 2     | Engine-committed chunk; utterance still open. Empty `SegmentFinal`s during online silence are "not speaking" heartbeats that keep the turn armed. Any other final closes the turn (hands-free): a non-empty phrase, or an empty offline/noise segment. |
| `UtteranceFinal` | 3     | Last frame for this utterance cycle; the handle always goes idle here. `text` holds the complete concatenated transcript. Auto-send (if configured) fires here once, on non-empty text.                                                                |

`BeginUtteranceRequest` fields:

| Field                    | Type   | Default | Meaning                                                                                                       |
| ------------------------ | ------ | ------- | ------------------------------------------------------------------------------------------------------------- |
| `agent_id`               | uint64 | `0`     | Auto-send target agent; `0` = transcribe-only.                                                                |
| `auto_finish_on_silence` | bool   | `true`  | Let server endpointing close each segment (hands-free). Ignored for PTT when the client calls `EndUtterance`. |
| `max_utterance_ms`       | uint32 | `60000` | Hard timeout per utterance segment.                                                                           |

Clients should update their display on `Partial` updates and commit on `UtteranceFinal`.

## Error responses

When the server cannot satisfy a request, it emits an `ErrorResponse` **instead of** the expected response frame:

| Field        | Meaning                                                                        |
| ------------ | ------------------------------------------------------------------------------ |
| `request_id` | Echoes the `request_id` of the failed request, if any.                         |
| `code`       | Numeric [error code](https://docs.tryllengine.com/reference/error-codes.html). |
| `message`    | Human-readable description; safe for display.                                  |

Inference errors that occur **during** a turn are not delivered as standalone `ErrorResponse` frames — they surface via `TurnComplete.status = Error`, with detail in `TurnComplete.debug_info` when diagnostics are on. Session-level and protocol-level errors still come through `ErrorResponse`.

Refer to [error codes](https://docs.tryllengine.com/reference/error-codes.html) for the full catalog and per-range recovery guidance.

## String storage

`CreateStringStorageRequest` creates a named string storage scoped to the current session. It carries four content fields:

| Field       | Type                           | Required                     | Description                                                                        |
| ----------- | ------------------------------ | ---------------------------- | ---------------------------------------------------------------------------------- |
| `name`      | string                         | yes                          | Unique session-scoped name.                                                        |
| `kind`      | `StringStorageKind` uint8 enum | no                           | `0 = List` (default), `1 = Map`, `2 = Multimap`.                                   |
| `strings`   | `[string]`                     | one of `strings`/`file_path` | Inline values. For Map/Multimap, parallel with `keys`.                             |
| `keys`      | `[string]`                     | only for Map/Multimap inline | Inline keys. Must have the same length as `strings`.                               |
| `file_path` | string                         | one of `strings`/`file_path` | Server-side file. `.txt` for List; `.json` array of `{id, text}` for Map/Multimap. |

**Backward compatibility:** old clients that omit `kind` receive `List` (0) as the default. Old clients that omit `keys` receive `null` (none), consistent with List creation. New fields are safely ignored by old server versions (FlatBuffers forwards-compatible table extension).

**Validation rules:**

- `kind = List`: `keys` must be absent or empty. `strings` **or** `file_path` required.
- `kind = Map`: `strings`+`keys` of equal non-zero length required (inline), **or** `file_path`. Duplicate keys are rejected (`7003`).
- `kind = Multimap`: same as Map but duplicate keys are accepted.

The server responds with `CreateStringStorageResponse` on success, or `ErrorResponse` on failure (see [error codes 7xxx](https://docs.tryllengine.com/reference/error-codes.html#7xxx-string-storage)).

## Inference throttle and agent workload

Added in **protocol v11**. The server and your game are separate processes sharing one GPU, and the server cannot see your frame budget. These two messages are how you tell it.

### `SetInferenceThrottle` — the manual knob

```
table SetInferenceThrottle {
  throttle: float = 0;   // 0 = full speed, 1 = maximum yielding
}
```

`0` means "run flat out" and is the default: **a session that never sends this message is never throttled**, so an existing integration behaves exactly as before. `1` means "yield as much as you can". Values in between scale linearly; out-of-range and non-finite values are ignored or clamped server-side and never produce an error.

Two properties make this message unusual:

- **No `request_id`, and no response.** It is the only client→server message besides [`AudioBuffer`](#voice-input-and-transcripts) with no reply of any kind — not even an `Ack`. Send it as often as you like from your load loop; there is nothing to correlate and nothing to await. A frame sent before `CreateSession` is silently dropped rather than answered with an error.
- **Session-scoped, but effective server-wide.** The server takes the **maximum** across all sessions that have reported, so the most-stressed client wins. A session's contribution is dropped when it disconnects.

The throttle does **not** change what is generated — the same prompt produces the same text — only how fast. The server spends the budget where it is cheapest: speech and background work slow first, visible streaming text is protected last.

```
# Drive from your frame loop, or on load transitions.
client.set_inference_throttle(0.0)   # full speed
client.set_inference_throttle(1.0)   # hand the GPU back
```

```
client.SetInferenceThrottle(0.0f);   // full speed
client.SetInferenceThrottle(1.0f);   // hand the GPU back
```

### `CreateAgentRequest.workload` — declaring background agents

```
enum AgentWorkload : uint8 { Interactive = 0, Background = 1 }
```

Slot 7 of `CreateAgentRequest`, defaulting to `Interactive`. Set it to `Background` for agents nobody is waiting on — world simulation, off-screen logic, evaluators, batch scoring. Background work is throttled hardest and yields to interactive work, which is what you want for it.

Everything else the server classifies on its own, per turn, from the agent's graph: a `Generate` node with `send = Streamed` is treated as visible text (gaps would be seen); `GenerateAndSpeak`, buffered `Generate`, `ToolCall` and `ClassifyIntentLLM` are treated as buffered (a gap is absorbed downstream before anyone notices). You do not declare any of that — and because `send` is mutable at runtime via `ChangeAgentParam`, the classification is re-derived every turn.

```
from tryll_client import AgentWorkload

agent = client.create_agent(graph, workload=AgentWorkload.Background)
```

```
Tryll::AgentCreateOptions opts;
opts.workload = Tryll::AgentWorkload::Background;
auto agent = client.CreateAgent(graph, opts);
```

Per-class tuning — chunk sizes, contention scaling, and the maximum pause — is server-side configuration, not part of the wire protocol. See [Server configuration](https://docs.tryllengine.com/reference/server-configuration.html#inference-scheduler).

## GraphDescription shape (protocol v2)

`CreateAgentRequest` carries a `GraphDescription` table with two fields:

| Field        | Type                | Description                                                                                                      |
| ------------ | ------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `nodes`      | `[NodeDescription]` | Ordered list of nodes. Each `NodeDescription` holds a name, a `params_type` union tag, and a typed params table. |
| `start_node` | `string`            | Name of the first node to execute each turn.                                                                     |

Wiring is encoded in the typed params table of **each source node**, not in a separate list. Every declared exit has a corresponding `<exit_name>_exit` string field (e.g. `default_exit`, `triggered_exit`). An empty string means *route to END*; any non-empty value must name another node in `nodes`.

The server validates all exit targets when processing `CreateAgentRequest`. A non-empty exit field that names a missing node is rejected with error [`3008 InvalidExitTarget`](https://docs.tryllengine.com/reference/error-codes.html#3xxx-agent).

## Versioning

The current wire-protocol version is **11**. Version 11 added the manual inference throttle — `SetInferenceThrottle` (56) — and `CreateAgentRequest.workload` (see [Inference throttle and agent workload](#inference-throttle-and-agent-workload)). Both are additive, but every client compares versions with `!=`, so any bump is a hard break at `ConnectionReady`. Version 10 was required because `RetrieveParams` gained `retrieval_mode` and `rrf_k` in the middle of the shipped FlatBuffers table (before `source` / `filter` / exits). Vtable slots are positional — mixed v9/v10 peers would decode those fields into the wrong slots — so the handshake hard-rejects a version mismatch. Version 9 replaced `ToolCallParams.notify_client`, `pause_after_tool_call`, and `history_policy` with a single `disposition: ToolCallDisposition` enum; `tool_call` NodeEvents now carry a `disposition` kv (not `history_policy`). Tool-call pause/resume and `tool_results` remain as introduced in v9 under the current v10 protocol. Version 8 extended `ResumeAgentRequest` with optional `tool_results: [ToolResult {call_id, result}]` for paused `AwaitResult` tool-call batches (see [Pausing and resuming a turn](#pausing-and-resuming-a-turn)). Version 7 added dialog history mutation: `AppendInteractionsRequest` / `AppendInteractionsResponse` (52/53) and `RemoveInteractionsFromEndRequest` / `RemoveInteractionsFromEndResponse` (54/55). Version 6 added agent KV-cache lifecycle initialization and the dedicated prefill, eviction, and status request/response pairs. Version 5 added [Variables](https://docs.tryllengine.com/concepts/agent-variables.html) messages (`UpdateAgentVariablesRequest`, `GetAgentVariablesRequest`, `GetAgentVariablesResponse`) and the trailing `CreateAgentRequest.variables` field; the version was bumped so a v5 client fails an incompatible v4 server at the `ConnectionReady` handshake rather than on the first variable request. Version 4 split the TCP connection from the logical session: `ConnectionReady` is now a connection-level hello with **no** `session_id`, and `CreateSession` (renamed from `ConfigureSession`) is mandatory and one-shot — it returns the server-allocated `session_id` in `CreateSessionResponse`. Every request before `CreateSession` is rejected with `2002 SessionNotReady`; a second `CreateSession` with `2003 SessionAlreadyExists`. (Version 2 had removed the `routes: [ExitRoute]` field from `GraphDescription` in favor of typed-param wiring; version 3 reinterpreted string-storage create/destroy as pin/unpin.)

When a breaking change lands, the server rejects incompatible clients at `ConnectionReady` time with error `5004` `ProtocolVersionMismatch`. Clients should surface the error message verbatim and stop reconnecting.

## Writing a new client: checklist

1. Open a TCP socket to the configured host/port.
1. Read exactly 4 bytes; interpret as little-endian uint32 → `length`.
1. Read exactly `length` bytes; decode as `Message` per `messages.fbs`.
1. Dispatch on the `body` union tag.
1. Read `ConnectionReady` first (unsolicited), then send `CreateSessionRequest`; block until `CreateSessionResponse` arrives.
1. Serialise per-agent `SendMessageRequest`s client-side; do not issue a second turn before `TurnComplete`.
1. Treat any frame > 1 MiB as a fatal protocol error.
1. Tolerate unsolicited frames (e.g. `NodeEvent`) interleaved with expected responses.

## Related

- [Error codes](https://docs.tryllengine.com/reference/error-codes.html) — numeric catalog.
- [Concept: Agents and sessions](https://docs.tryllengine.com/concepts/agents-and-sessions.html) — state-machine view of the lifecycle above.
- `tryll/schema/messages.fbs` — canonical schema.
