# Wire Protocol

This page specifies the byte-level contract a third-party client must implement to talk to a Tryll server without using the shipped C++, Python, or Unreal client libraries. If you are using one of those libraries, you do not need any of this — skip to the [how-to guides](https://docs.tryllengine.com/v0.9/how-to/index.html).

The authoritative source of truth for every structure named below is the FlatBuffers schema file `tryll/schema/messages.fbs`. The current **protocol version is 27** (sent in `ConnectionReady.protocol_version`). This reference describes the transport around that schema. `ConnectionReady` also carries `mode` (deployment mode string).

## Transport

- **TCP** only. No TLS; the server is expected to run on `localhost` or inside a trusted network.
- **Bi-directional**. Both client-to-server requests and server-to-client responses use the same framing over the same socket.
- **One connection = one [session](https://docs.tryllengine.com/v0.9/reference/glossary.html#session).** Closing the socket destroys every [agent](https://docs.tryllengine.com/v0.9/reference/glossary.html#agent) owned by the session.

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
- **Maximum frame size: 1 MiB (1 048 576 bytes).** The cap is a fixed constant, not negotiated. When the server reads a `length` of `0` or above the cap, it closes the connection without sending an `ErrorResponse`. The server answers with `5003` `FrameTooLarge` (see [error codes](https://docs.tryllengine.com/v0.9/reference/error-codes.html#5xxx-protocol-framing)) only when a response it built would not fit in one frame. Unity and Unreal detect an oversized *outbound* frame before writing it, fail that one request (`5003`, or `3021` for ImportDialog), and keep the connection.

No compression, no chunking below the FlatBuffers level, no keep-alive heartbeats. The client is expected to read frames in a loop and dispatch them by the `body` union tag.

## Message types

All messages are defined in `messages.fbs` under the `MessageBody` union. The full catalog (table names as they appear in the schema):

| Name                                  | Kind            | Direction       | Returns                                          | Summary                                                                                                                                             |
| ------------------------------------- | --------------- | --------------- | ------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ConnectionReady`                     | Notification    | Server → Client | —                                                | Sent immediately after a connection is accepted (connection-level hello).                                                                           |
| `CreateSessionRequest`                | Request         | Client → Server | `CreateSessionResponse`                          | Create the logical session for this connection (mandatory, one-shot).                                                                               |
| `CreateSessionResponse`               | Response        | Server → Client | —                                                | Response to CreateSessionRequest.                                                                                                                   |
| `CreateAgentRequest`                  | Request         | Client → Server | `CreateAgentResponse`                            | Create an agent from a workflow graph and bind its variables.                                                                                       |
| `CreateAgentResponse`                 | Response        | Server → Client | —                                                | Returns the server-allocated agent_id for CreateAgentRequest.                                                                                       |
| `SendMessageRequest`                  | Request         | Client → Server | `TurnComplete` (progress `AnswerText`)           | Start a turn: the user text for one agent.                                                                                                          |
| `DestroyAgentRequest`                 | Request         | Client → Server | `Ack`                                            | Destroy an agent.                                                                                                                                   |
| `AnswerText`                          | Streaming       | Server → Client | —                                                | Streaming answer chunk.                                                                                                                             |
| `TurnComplete`                        | Streaming       | Server → Client | —                                                | Terminal message for a SendMessageRequest.                                                                                                          |
| `ErrorResponse`                       | Response        | Server → Client | —                                                | Protocol or application error.                                                                                                                      |
| `Ack`                                 | Response        | Server → Client | —                                                | Confirms a request that has no dedicated response table (DestroyAgent, ChangeAgentParam, UnloadModel, Destroy\*Storage, VoiceInput control ops, …). |
| `ListModelsRequest`                   | Request         | Client → Server | `ListModelsResponse`                             | Request a list of all models in the server's model catalog.                                                                                         |
| `ListModelsResponse`                  | Response        | Server → Client | —                                                | Response to ListModelsRequest.                                                                                                                      |
| `DownloadModelRequest`                | Request         | Client → Server | `DownloadComplete` (progress `DownloadProgress`) | Initiate a download for the named catalog model (its entry names the engine).                                                                       |
| `DownloadProgress`                    | Streaming       | Server → Client | —                                                | Streaming progress for an active download.                                                                                                          |
| `DownloadComplete`                    | Response        | Server → Client | —                                                | Terminal message for a DownloadModelRequest.                                                                                                        |
| `CreateStringStorageRequest`          | Request         | Client → Server | `CreateStringStorageResponse`                    | Pin a StringStorage into the session storage manager, scoped to this session.                                                                       |
| `CreateStringStorageResponse`         | Response        | Server → Client | —                                                | Response to CreateStringStorageRequest.                                                                                                             |
| `DestroyStringStorageRequest`         | Request         | Client → Server | `Ack`                                            | Unpin (or unregister) a StringStorage.                                                                                                              |
| `NodeEvent`                           | Notification    | Server → Client | —                                                | Unsolicited generic event fired by a workflow node when its notify_client flag is set.                                                              |
| `LoadModelRequest`                    | Request         | Client → Server | `LoadModelResponse`                              | Explicitly load and pin a catalog model into memory on the engine its entry names.                                                                  |
| `LoadModelResponse`                   | Response        | Server → Client | —                                                | Confirms a LoadModelRequest, and reports what actually happened.                                                                                    |
| `UnloadModelRequest`                  | Request         | Client → Server | `Ack`                                            | Unpin a previously pinned model.                                                                                                                    |
| `CreateEmbeddedStringStorageRequest`  | Request         | Client → Server | `CreateEmbeddedStringStorageResponse`            | Pin an EmbeddedStringStorage into the session storage manager, scoped to this session.                                                              |
| `CreateEmbeddedStringStorageResponse` | Response        | Server → Client | —                                                | Response to CreateEmbeddedStringStorageRequest.                                                                                                     |
| `DestroyEmbeddedStringStorageRequest` | Request         | Client → Server | `Ack`                                            | Unpin (Path A) or unregister (Path B) an EmbeddedStringStorage.                                                                                     |
| `ChangeAgentParamRequest`             | Request         | Client → Server | `Ack`                                            | Mutate all typed parameters on a named node within an existing agent's workflow.                                                                    |
| `CreateVoiceInputRequest`             | Request         | Client → Server | `CreateVoiceInputResponse`                       | Create a persistent VoiceInput handle on the server.                                                                                                |
| `CreateVoiceInputResponse`            | Response        | Server → Client | —                                                | Response; carries the voice_input_id used by subsequent frames.                                                                                     |
| `BeginUtteranceRequest`               | Request         | Client → Server | `Ack`                                            | Open a new utterance on the handle.                                                                                                                 |
| `AudioBuffer`                         | Fire-and-forget | Client → Server | —                                                | Streaming audio frame (no request_id; correlated by voice_input_id).                                                                                |
| `EndUtteranceRequest`                 | Request         | Client → Server | `Ack`                                            | Commit the current utterance (push-to-talk release).                                                                                                |
| `CancelUtteranceRequest`              | Request         | Client → Server | `Ack`                                            | Cancel in-flight partial; no transcript.                                                                                                            |
| `WireTranscriptUpdate`                | Notification    | Server → Client | —                                                | Server-pushed transcript update for one voice input.                                                                                                |
| `DestroyVoiceInputRequest`            | Request         | Client → Server | `Ack`                                            | Release the persistent handle.                                                                                                                      |
| `TtsAudioFormatFrame`                 | Streaming       | Server → Client | —                                                | Sent once per agent turn, by the first speaking node, before the turn's first TtsAudioFrame.                                                        |
| `TtsAudioFrame`                       | Streaming       | Server → Client | —                                                | One chunk of synthesized PCM for the current turn.                                                                                                  |
| `DeleteModelRequest`                  | Request         | Client → Server | `Ack`                                            | Delete a downloaded model's files from disk and drop its downloads.json entry.                                                                      |
| `DebugCommandRequest`                 | Request         | Client → Server | `DebugCommandResponse`                           | Dev-only. Generic developer/diagnostic command channel.                                                                                             |
| `DebugCommandResponse`                | Response        | Server → Client | —                                                | Dev-only reply from a debug command that does not fault the server.                                                                                 |
| `CancelRequest`                       | Request         | Client → Server | `Ack`                                            | Cooperatively cancel the agent's in-flight turn.                                                                                                    |
| `ResumeAgentRequest`                  | Request         | Client → Server | `Ack`                                            | Resume a turn that is currently paused (see PauseNode, ToolCallDisposition Pause / PauseAndAcknowledge / AwaitResult).                              |
| `UpdateAgentVariablesRequest`         | Request         | Client → Server | `Ack`                                            | Batched mutation of an existing agent's declared variables.                                                                                         |
| `GetAgentVariablesRequest`            | Request         | Client → Server | `GetAgentVariablesResponse`                      | Debug/verification read of an agent's current variable values.                                                                                      |
| `GetAgentVariablesResponse`           | Response        | Server → Client | —                                                | Response to GetAgentVariablesRequest.                                                                                                               |
| `PrefillAgentKvCacheRequest`          | Request         | Client → Server | `PrefillAgentKvCacheResponse`                    | Synchronise every eligible language-model node to the reusable cacheable prefix.                                                                    |
| `PrefillAgentKvCacheResponse`         | Response        | Server → Client | —                                                | Aggregate result of PrefillAgentKvCacheRequest.                                                                                                     |
| `EvictAgentKvCacheRequest`            | Request         | Client → Server | `EvictAgentKvCacheResponse`                      | Release every eligible language-model node's raw backend context while preserving the agent, graph, dialogue, and projection state.                 |
| `EvictAgentKvCacheResponse`           | Response        | Server → Client | —                                                | Aggregate result of EvictAgentKvCacheRequest.                                                                                                       |
| `GetAgentKvCacheStatusRequest`        | Request         | Client → Server | `GetAgentKvCacheStatusResponse`                  | Read-only aggregate KV-cache residency / prefix status for an agent.                                                                                |
| `GetAgentKvCacheStatusResponse`       | Response        | Server → Client | —                                                | Aggregate result of GetAgentKvCacheStatusRequest.                                                                                                   |
| `AppendInteractionsRequest`           | Request         | Client → Server | `AppendInteractionsResponse`                     | Append scripted user/assistant interactions without running the graph.                                                                              |
| `AppendInteractionsResponse`          | Response        | Server → Client | —                                                | How many scripted interactions were appended.                                                                                                       |
| `RemoveInteractionsFromEndRequest`    | Request         | Client → Server | `RemoveInteractionsFromEndResponse`              | Remove whole interactions from the end of the dialog.                                                                                               |
| `RemoveInteractionsFromEndResponse`   | Response        | Server → Client | —                                                | How many interactions were removed from the end of the dialog.                                                                                      |
| `SetInferenceThrottle`                | Fire-and-forget | Client → Server | —                                                | The game reports how hard the server should yield the GPU.                                                                                          |
| `SetSessionLocaleRequest`             | Request         | Client → Server | `Ack`                                            | Change the session language after CreateSession, without tearing the session down.                                                                  |
| `ExportDialogRequest`                 | Request         | Client → Server | `ExportDialogResponse`                           | Export one agent dialog as a persisted snapshot.                                                                                                    |
| `ExportDialogResponse`                | Response        | Server → Client | —                                                | Persisted dialog snapshot for an ExportDialogRequest.                                                                                               |
| `ImportDialogRequest`                 | Request         | Client → Server | `ImportDialogResponse`                           | Restore one agent dialog from a persisted snapshot.                                                                                                 |
| `ImportDialogResponse`                | Response        | Server → Client | —                                                | How many interactions were restored, plus any compatibility warnings.                                                                               |
| `GetMemoryConsumptionRequest`         | Request         | Client → Server | `GetMemoryConsumptionResponse`                   | Pull a server-wide memory snapshot.                                                                                                                 |
| `GetMemoryConsumptionResponse`        | Response        | Server → Client | —                                                | The server-wide memory snapshot for GetMemoryConsumptionRequest.                                                                                    |
| `EnsureMutableKbRequest`              | Request         | Client → Server | `EnsureMutableKbResponse`                        | Create or reuse a named mutable knowledge base for a schema.                                                                                        |
| `EnsureMutableKbResponse`             | Response        | Server → Client | —                                                | The store name, schema, and whether EnsureMutableKb created it.                                                                                     |
| `EnumerateMutableKbsRequest`          | Request         | Client → Server | `EnumerateMutableKbsResponse`                    | List the session mutable knowledge bases.                                                                                                           |
| `EnumerateMutableKbsResponse`         | Response        | Server → Client | —                                                | One MutableKbInfo row per store in the session.                                                                                                     |
| `MutableKbWriteRequest`               | Request         | Client → Server | `MutableKbWriteResponse`                         | Upsert records into a mutable knowledge base.                                                                                                       |
| `MutableKbWriteResponse`              | Response        | Server → Client | —                                                | Per-record result of a MutableKbWriteRequest.                                                                                                       |
| `MutableKbDeleteRequest`              | Request         | Client → Server | `MutableKbDeleteResponse`                        | Delete records from a mutable knowledge base by id.                                                                                                 |
| `MutableKbDeleteResponse`             | Response        | Server → Client | —                                                | How many records a MutableKbDeleteRequest removed.                                                                                                  |
| `MutableKbClearRequest`               | Request         | Client → Server | `MutableKbClearResponse`                         | Delete every record in a mutable knowledge base.                                                                                                    |
| `MutableKbClearResponse`              | Response        | Server → Client | —                                                | How many records a MutableKbClearRequest removed.                                                                                                   |
| `MutableKbQueryRequest`               | Request         | Client → Server | `MutableKbQueryResponse`                         | Nearest-neighbour query against a mutable knowledge base.                                                                                           |
| `MutableKbQueryResponse`              | Response        | Server → Client | —                                                | Hits for a MutableKbQueryRequest, best first.                                                                                                       |
| `MutableKbGetRequest`                 | Request         | Client → Server | `MutableKbGetResponse`                           | Fetch one record from a mutable knowledge base by id.                                                                                               |
| `MutableKbGetResponse`                | Response        | Server → Client | —                                                | The record for a MutableKbGetRequest, or found=false.                                                                                               |
| `MutableKbListRequest`                | Request         | Client → Server | `MutableKbListResponse`                          | Page through the records of a mutable knowledge base.                                                                                               |
| `MutableKbListResponse`               | Response        | Server → Client | —                                                | One page of records from a MutableKbListRequest.                                                                                                    |
| `MutableKbStatsRequest`               | Request         | Client → Server | `MutableKbStatsResponse`                         | Read size and materialization stats for a mutable knowledge base.                                                                                   |
| `MutableKbStatsResponse`              | Response        | Server → Client | —                                                | Live, tombstoned, and byte counts for a mutable knowledge base.                                                                                     |
| `ExportMutableKbRequest`              | Request         | Client → Server | `ExportMutableKbResponse`                        | Export one mutable knowledge base as a persisted snapshot.                                                                                          |
| `ExportMutableKbResponse`             | Response        | Server → Client | —                                                | The persisted snapshot for an ExportMutableKbRequest.                                                                                               |
| `ImportMutableKbRequest`              | Request         | Client → Server | `ImportMutableKbResponse`                        | Import a persisted mutable knowledge base into the session.                                                                                         |
| `ImportMutableKbResponse`             | Response        | Server → Client | —                                                | The imported store name and any compatibility warnings.                                                                                             |
| `SaveGameMemoryRequest`               | Request         | Client → Server | `SaveGameMemoryResponse`                         | Write every mutable knowledge base to a game-memory file.                                                                                           |
| `SaveGameMemoryResponse`              | Response        | Server → Client | —                                                | When the game-memory file was written, and how many stores it holds.                                                                                |
| `LoadGameMemoryRequest`               | Request         | Client → Server | `LoadGameMemoryResponse`                         | Replace the session mutable knowledge bases from a game-memory file.                                                                                |
| `LoadGameMemoryResponse`              | Response        | Server → Client | —                                                | How many stores LoadGameMemory restored, plus compatibility warnings.                                                                               |
| `SaveDialogRequest`                   | Request         | Client → Server | `SaveDialogResponse`                             | Write one agent dialog snapshot to a file the game owns.                                                                                            |
| `SaveDialogResponse`                  | Response        | Server → Client | —                                                | When the dialog file was written.                                                                                                                   |
| `LoadDialogRequest`                   | Request         | Client → Server | `LoadDialogResponse`                             | Restore one agent dialog from a file the game owns.                                                                                                 |
| `LoadDialogResponse`                  | Response        | Server → Client | —                                                | How many interactions LoadDialog restored, plus compatibility warnings.                                                                             |
| `TtsVisemeFrame`                      | Streaming       | Server → Client | —                                                | Viseme timings for one synthesized sentence, sent after that sentence audio.                                                                        |
| `SetAgentVoiceRequest`                | Request         | Client → Server | `Ack`                                            | Replace an agent's voice (protocol v27).                                                                                                            |
| `SetAgentDeliveryModeRequest`         | Request         | Client → Server | `Ack`                                            | Switch an agent's active delivery mode (protocol v27).                                                                                              |

Model requests are not tied to a per-session engine. Each catalog entry names its own engine: `ListModelsRequest` returns every non-hidden catalog model with that engine as data, and `DownloadModelRequest` and `LoadModelRequest` use the engine the catalog entry names. See [Session lifecycle](#session-lifecycle).

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
    C->>S: CreateSessionRequest(model_config: engines + placement_hints)
    S-->>C: CreateSessionResponse
    C->>S: CreateAgentRequest(graph, voice?, ...)
    note over S: fails fast (GraphCompilationFailed / InvalidVoiceProfile) if a referenced model is absent
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

`CreateSessionRequest` carries a `model_config` describing what the session needs from the model layer:

- **`engines`** — an **advisory** hint naming the `(role, engine)` pairs the session expects to touch, e.g. language = `llama-cpp`, tts = `sherpa-onnx`. The catalog already records which engine runs each model, so this gates nothing: it may be empty, incomplete, or wrong, and loading a model it never named works normally. Its one effect is to let the server initialise those engines' backends when the session is created rather than on first use, which matters when that cost is large. A model whose engine is absent from the *server build* is refused at load with `EngineNotAvailable` (6008), naming the model, its role and the engine. Only an out-of-range enum value is rejected outright, with `InvalidEngineSet` (2005).
- **`placement_hints`** — optional per-model `MemoryPlacement` (`auto` / `ram` / `vram`). Sparse: list only the models you have an opinion about. These are hints, not guarantees — see [Choose where models run](https://docs.tryllengine.com/v0.9/how-to/choose-where-models-run.html).

VAD is catalog/download-only — `LoadModelRequest` for a VAD model returns `ModelResolutionFailed`. A VAD model needs no declaration of its own: it loads as a dependency of an STT model and rides that declaration.

> Before protocol v13 this table carried four engine fields (`inference_engine`, `stt_engine`, `tts_engine`, `embedding_engine`), three defaulting to `Mock`. `ListModels` also filtered by them, which meant a session that left them at their defaults saw no STT, TTS or embedding models at all. `ListModels` now reports every catalog model with its own engine as data.

The storage root is **process-wide** and fixed at server startup (`--storage-root` / `storage_root` / mode default). Protocol v24 removed `CreateSessionRequest.storage_data_folder`. Relative paths in node `string_storage` and `embedded_string_storage` params resolve against that root. See [Server Configuration](https://docs.tryllengine.com/v0.9/reference/server-configuration.html#storage-root) and [Ship Storage Folders for Builds](https://docs.tryllengine.com/v0.9/how-to/ship-storage-folders-for-builds.html).

It also carries `locale: string` — the session language as a BCP-47 tag (`"de"`, `"pt-BR"`). One setting the whole voice + text stack derives from: the default `lang` of every agent's voice profile, the language hint handed to multilingual STT models, the `loc/<language>/` folder beside a linked file that storage paths prefer when a translation exists, and the `locale` / `language` / `language_code` Mustache keys. Empty leaves every one of those at its pre-locale behaviour; a malformed tag is rejected with `InvalidLocale` (2004). `SetSessionLocaleRequest` (57) changes it later without tearing the session down — same grammar, same rejection, and an empty tag clears it. See [Ship an Agent in Another Language](https://docs.tryllengine.com/v0.9/how-to/ship-in-another-language.html).

`CreateVoiceInputRequest` now carries `hotwords_storage_path: string` — a relative path (resolved against the process storage root) to a plain `.txt` file listing hotwords, one per line, and `hotwords_score: float` (the bonus added to each matched hotword phrase during beam search, weighting phrases against one another; default `1.5`). The score is ignored when `hotwords_storage_path` is empty, and a value that is not a finite, non-negative number is rejected with `InvalidHotwordsScore` (4107). These are exposed as `HotwordsStoragePath` and `HotwordsScore` UPROPERTYs on the Unreal voice component.

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

Both operations require a **strictly idle** agent: rejected with `3004 AgentBusy` while a turn is running, while a turn is **paused**, or while a KV-cache lifecycle operation is in flight. This is stricter than [`ChangeAgentParam`](https://docs.tryllengine.com/v0.9/how-to/change-agent-parameters.html), which is allowed while paused.

Either mutation invalidates the agent's reusable KV prefix (`_kvCachePrefixCurrent = false`). Removal additionally resets the token-budget projection window on every language-model context. The next `SendMessage` pays re-sync / re-prefill cost for the edited suffix.

**Stateless agents** (`maintain_dialogue_history = false`) may append scripted history as a one-shot seed before a single `SendMessage`; the dialogue is cleared after that turn completes, as with any other turn.

**vs `StopAndDiscard`.** [`Cancel`](https://docs.tryllengine.com/v0.9/how-to/cancel-a-turn.html) with `StopAndDiscard` removes the **last in-flight** interaction after a cooperative cancel mid-turn. Dialog mutation edits completed or injected history on an idle agent; tail removal is explicit and count-based.

The Unity (`TryllAgent.AppendInteractions` / `RemoveInteractionsFromEnd`), Unreal (`FTryllAgent::AppendInteractions` / `RemoveInteractionsFromEnd`), C++ (`AgentProxy::AppendInteractions` / `RemoveInteractionsFromEnd`), and Python (`AgentProxy.append_interactions` / `remove_interactions_from_end`) clients expose these directly. See [Seed and edit dialog history](https://docs.tryllengine.com/v0.9/how-to/seed-and-edit-dialog-history.html).

### ExportDialog / ImportDialog

Protocol v14 adds idle-only export/import of a **full-fidelity** dialog snapshot. Tryll does not write files — the game owns the outer save container.

`ExportDialogRequest {request_id, agent_id}`

`ExportDialogResponse {request_id, agent_id, snapshot}` where `snapshot` is a `PersistedDialogSnapshot` (also a standalone root, file identifier `TDLG`). `format_version` is independent of the wire protocol.

`ImportDialogRequest {request_id, agent_id, snapshot, compatibility}`

`compatibility`: `Strict` (default) or `AllowGraphMismatch`.

`ImportDialogResponse {request_id, agent_id, interaction_count, warnings}`

Import **replaces** the dialog. Failures are atomic. Encoded snapshots that would overflow the 1 MiB frame fail with `DialogSnapshotTooLarge` (3021). Unity and Unreal also reject that case locally so the connection stays up. See [Export and import dialog](https://docs.tryllengine.com/v0.9/how-to/export-and-import-dialog.html).

## Streaming answers

For every [turn](https://docs.tryllengine.com/v0.9/reference/glossary.html#turn), the server emits **one or more** `AnswerText` frames followed by exactly one `TurnComplete`:

| Field       | Meaning                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `agent_id`  | Echoes the target agent.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `node_name` | Name of the workflow node that produced this chunk (multi-sender attribution — a graph can route through more than one text-producing node in a single turn, e.g. two `CannedResponse` nodes voicing two NPCs). Clients that only care about the text may ignore this field.                                                                                                                                                                                                                                             |
| `text`      | The text chunk.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `is_delta`  | `true` when `text` holds only the new text, to append to what the node already sent. A `send = Streamed` node sends every chunk with `is_delta = true`, and its final chunk is an empty terminator (`is_delta = true`, `is_final = true`, `text = ""`). A `send = Whole` node sends one chunk with `is_delta = false`, `is_final = true`, and the full text.                                                                                                                                                             |
| `is_final`  | `true` on the last chunk from a given node. Each text-producing node emits its own final chunk for its `node_name`; a turn with a single producing node still has exactly one `is_final = true` chunk overall, but a turn that routes through multiple producing nodes (or a node whose `send = None`, which never emits) will have one `is_final = true` per node that emitted. A turn that produces no text at all (e.g. routed to a canned response with empty output) still emits one final chunk with empty `text`. |

`TurnComplete` carries:

| Field              | Meaning                                                                                                                                                                                                        |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `agent_id`         | Echoes the target agent.                                                                                                                                                                                       |
| `status`           | `TurnStatus::Success` (0), `Error` (1), `Cancelled` (2), or `PauseTimedOut` (3) — a pause outlived its timeout with no timeout exit to take (see [Pausing and resuming a turn](#pausing-and-resuming-a-turn)). |
| `debug_info`       | JSON string with per-node execution data; populated only when the agent was created with `enable_diagnostics = true`. Empty string otherwise.                                                                  |
| `tokens_generated` | Total tokens sampled across all generation nodes in this turn (prompt tokens excluded).                                                                                                                        |

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

- a [`Pause`](https://docs.tryllengine.com/v0.9/reference/nodes/pause.html) node — a no-op checkpoint; the executor always pauses after it,
- [`ToolCall`](https://docs.tryllengine.com/v0.9/reference/nodes/tool-call.html) with a pausing `disposition` (`Pause`, `PauseAndAcknowledge`, `AwaitResult`) — pauses only when a tool call was detected (the `tool_called` exit).

While paused, the turn is still open (`SendMessage` still gets `AgentBusy`), but [`ChangeAgentParam`](https://docs.tryllengine.com/v0.9/how-to/change-agent-parameters.html) — normally rejected while a turn is running — is allowed. This is the point of pausing: react to a tool call (or any checkpoint) by mutating node params before the graph continues.

A pause can time out:

- **Pause node** — `pause_timeout_ms` `0` (the default) waits forever. With a timeout, expiry takes `timeout_exit` if one is set; an empty `timeout_exit` ends the turn with `TurnStatus.PauseTimedOut`.
- **ToolCall** — `pause_timeout_ms` `0` inherits the server's `workflow.tool_call_pause_timeout_ms` (see [Server Configuration](https://docs.tryllengine.com/v0.9/reference/server-configuration.html#workflow-workflow)), which is **2000 ms** in the shipped `server-config.json`. ToolCall has no timeout exit: expiry ends the turn with `TurnStatus.PauseTimedOut`.

A pause with no deadline lasts until the client sends `ResumeAgentRequest` or `CancelRequest`.

The server announces a pause with a `NodeEvent` (`event_type = "paused"`) whose `kv_pairs` are `exit_route`, `cause` (`tool_call` or `pause_node`), and `timeout_ms` (the pause deadline in milliseconds; `0` means no deadline). It uses the same fire-and-forget, unsolicited channel as `tool_call` and `intent_classified` (see [Tool Call](https://docs.tryllengine.com/v0.9/reference/nodes/tool-call.html)). Protocol v8 added `ResumeAgentRequest.tool_results`; protocol v9 consolidates ToolCall notify / pause / history into `disposition`. Resumes use `ResumeAgentRequest {agent_id, resume_node, tool_results[]}`:

| `resume_node` | Effect                                                                                                                            |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Empty         | Continue via the paused node's wired exit route (the `exit_route` from the `paused` event).                                       |
| Non-empty     | Jump directly to that node by name, skipping the wired exit. `ErrorResponse(UnknownNode, 3005)` if it doesn't exist in the graph. |

Each optional `ToolResult` is `{call_id, result}`; `call_id` must match the stable id from the `tool_call` event. A non-empty batch must contain exactly one result for every pending `AwaitResult` call. The server validates the whole batch and attaches all results before waking the turn. Unknown, duplicate, missing, non-tool-pause, or oversized batches return `ErrorResponse(InvalidToolResults, 3016)` and leave the turn paused for retry. An empty batch preserves plain continue behavior.

`ResumeAgentRequest` while the agent is idle, or actively running (not paused), returns `ErrorResponse(AgentNotPaused, 3012)`.

Clients expose plain continue/jump via `Resume` / `resume`, and tool-result batches via `ResumeWithToolResult(s)` (Unity / Unreal / C++) or `resume(..., tool_results=)` (Python). Correlation ids arrive on `ToolCallNotificationWithId` / `OnToolCallWithId` / `SetOnToolCallWithId` / `set_on_tool_call_with_id` (legacy name-only callbacks still fire). Unity and Unreal also offer `RegisterTool` sugar that auto-resumes a complete paused batch when every call has a registered handler. Auto-resume attaches `tool_results` only for `await_result`; acknowledgement / route-only dispositions resume payload-free. A payload-free Resume while `AwaitResult` calls are still pending is rejected with `3016 InvalidToolResults`. Typed `paused` callbacks: `SetOnPaused` / `set_on_paused` / Unity `Paused` / Unreal `OnPaused`. See [How to pause and resume a turn](https://docs.tryllengine.com/v0.9/how-to/pause-and-resume-turns.html) and [Define and handle tool calls](https://docs.tryllengine.com/v0.9/how-to/define-and-handle-tool-calls.html).

## Agent Variables

`CreateAgentRequest` carries a trailing `variables: [AgentVariable]` field — the full, type-locked declaration of the agent's [Variables](https://docs.tryllengine.com/v0.9/concepts/agent-variables.html) store, each entry a `{name: string, value: VariableValue}` pair. `VariableValue` is a union over `VarInt` (int64), `VarFloat` (float64), `VarString`, `VarBool`, and `VarStringSet` (`[string]`).

`UpdateAgentVariablesRequest {agent_id, request_id?, updates: [VariableUpdate]}` applies a batch of mutations atomically — every update is validated before any is applied, so a rejected batch never partially mutates the store. Each `VariableUpdate` is `{name: string, op: VariableOp, value: VariableValue?}`:

| `op`            | Requires `value`?   | Effect                                                                    |
| --------------- | ------------------- | ------------------------------------------------------------------------- |
| `Assign`        | yes                 | Replace the variable's current value.                                     |
| `AddToSet`      | yes, `VarString`    | Insert a string into a `set<string>` variable (no-op if already present). |
| `RemoveFromSet` | yes, `VarString`    | Remove a string from a `set<string>` variable (no-op if already absent).  |
| `Reset`         | no (must be absent) | Restore the `CreateAgent`-time initial value.                             |

The server replies `Ack` on success, or `ErrorResponse` on the first invalid update in the batch (`3013 UnknownVariable`, `3014 VariableTypeMismatch`, `3015 InvalidVariableValue` for an unknown op, a missing or over-limit value, or a set that would exceed 4096 elements, or `3004 AgentBusy` — admission mirrors [`ChangeAgentParam`](https://docs.tryllengine.com/v0.9/how-to/change-agent-parameters.html): allowed while idle or paused, rejected while a turn is actively running).

`GetAgentVariablesRequest {agent_id, request_id?}` returns `GetAgentVariablesResponse {request_id?, variables: [AgentVariable]}` — a full snapshot of the agent's current values. This response carries no `agent_id`; correlate multiple in-flight `Get` calls via `request_id`.

Declared variables are consumed server-side in two places: Mustache templates via `{{var.<name>}}` (see [Use Mustache templates](https://docs.tryllengine.com/v0.9/how-to/use-mustache-templates.html)) and the Retrieve/ClassifyIntent filter grammar's `{"var": "<name>"}` operand (see [Retrieve filter grammar](https://docs.tryllengine.com/v0.9/reference/nodes/retrieve-filter-grammar.html)) — both re-read the current value on every use, so a mutation takes effect on the very next turn with no recompile.

## Agent voice

Since protocol v27 an agent's voice is one `VoiceProfile` (schema `tryll/schema/voice.fbs`), not a set of fields on each speaking node. See [Voice Profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html) for every field and rule.

- **`CreateAgentRequest.voice: VoiceProfile`.** Required when any node in the graph speaks (`SpeakParams`, `GenerateAndSpeakParams`); without it creation fails with `3025 AgentVoiceRequired`. A profile that does not fit its model fails with `3023 InvalidVoiceProfile`. When no node speaks, the voice is stored and no TTS model is loaded.
- **`SetAgentVoiceRequest {request_id, agent_id, voice}` → `Ack`.** Replaces the voice. Admitted only while the agent is idle: a running or paused turn gets `3004 AgentBusy`. The new profile is validated and its clips prepared before the swap; on failure the old voice stays. The active delivery mode is kept if the new profile defines a mode of the same name, otherwise the new profile's `default_delivery_mode` applies and `Ack.warning` says so. Every other ack leaves `warning` empty.
- **`SetAgentDeliveryModeRequest {request_id, agent_id, mode, intensity = 1.0}` → `Ack`.** Switches the active delivery mode. Admitted at any time, including mid-turn, and applies from the next sentence handed to the TTS model. `mode` is a mode name from the profile or `neutral` (empty also means `neutral`); an unknown name gets `3024 UnknownDeliveryMode` and the current mode stays. `intensity` is clamped to 0..1.
- Either request on an agent created without a voice gets `3025`.
- **One audio stream per turn.** `TtsAudioFormatFrame` is sent once per agent turn, by the first speaking node that produces audio; later speaking nodes reuse it. `TtsVisemeFrame` sample offsets count from it across every speaking node of the turn. `TtsAudioFrame.node_name` still names the node that produced each chunk.
- **`TtsCapabilitiesInfo`** (in `ListModelsResponse`) gains `expressiveness: bool` and `num_speakers: int` (`0` = not advertised), so an editor can show only the profile fields a model accepts.
- **`ChangeAgentParamRequest`** can no longer reach any voice field. Speaking nodes keep only `min_sentence_chars` and `first_chunk_max_chars` as mutable TTS settings.

## Voice input and transcripts

When a VoiceInput utterance is active the server streams `WireTranscriptUpdate` frames to the client. Each frame carries a `kind` field that classifies the event:

| `kind`           | Value | Meaning                                                                                                                                                                                                                                                |
| ---------------- | ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `SpeechStart`    | 0     | VAD rising edge; `text` is empty.                                                                                                                                                                                                                      |
| `Partial`        | 1     | Revisable in-progress hypothesis. Overwrite any previous partial.                                                                                                                                                                                      |
| `SegmentFinal`   | 2     | Engine-committed chunk; utterance still open. Empty `SegmentFinal`s during online silence are "not speaking" heartbeats that keep the turn armed. Any other final closes the turn (hands-free): a non-empty phrase, or an empty offline/noise segment. |
| `UtteranceFinal` | 3     | Last frame for this utterance cycle; the handle always goes idle here. `text` holds the complete concatenated transcript. The client starts any turn from that text.                                                                                   |

`BeginUtteranceRequest` fields:

| Field                    | Type   | Default | Meaning                                                                                                       |
| ------------------------ | ------ | ------- | ------------------------------------------------------------------------------------------------------------- |
| `auto_finish_on_silence` | bool   | `true`  | Let server endpointing close each segment (hands-free). Ignored for PTT when the client calls `EndUtterance`. |
| `max_utterance_ms`       | uint32 | `60000` | Hard timeout per utterance segment.                                                                           |

Clients should update their display on `Partial` updates and commit on `UtteranceFinal`.

## Error responses

When the server cannot satisfy a request, it emits an `ErrorResponse` **instead of** the expected response frame:

| Field        | Meaning                                                                             |
| ------------ | ----------------------------------------------------------------------------------- |
| `request_id` | Echoes the `request_id` of the failed request, if any.                              |
| `code`       | Numeric [error code](https://docs.tryllengine.com/v0.9/reference/error-codes.html). |
| `message`    | Human-readable description; safe for display.                                       |

A turn that fails while it runs is reported twice: first an `ErrorResponse` with `4001` `InferenceFailed` carrying the human-readable message (for example, an exceeded step limit arrives as "Max steps per turn (…) exceeded"), then `TurnComplete` with `status = Error`. The `ErrorResponse` is sent whether or not diagnostics are on, so do not discard `ErrorResponse` frames that arrive during a turn. Session-level and protocol-level errors also come through `ErrorResponse`.

Refer to [error codes](https://docs.tryllengine.com/v0.9/reference/error-codes.html) for the full catalog and per-range recovery guidance.

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

The server responds with `CreateStringStorageResponse` on success, or `ErrorResponse` on failure (see [error codes 7xxx](https://docs.tryllengine.com/v0.9/reference/error-codes.html#7xxx-string-storage)).

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

Per-class tuning — chunk sizes, contention scaling, and the maximum pause — is server-side configuration, not part of the wire protocol. See [Server configuration](https://docs.tryllengine.com/v0.9/reference/server-configuration.html#inference-scheduler).

## GraphDescription shape (protocol v2)

`CreateAgentRequest` carries a `GraphDescription` table with two fields:

| Field        | Type                | Description                                                                                                      |
| ------------ | ------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `nodes`      | `[NodeDescription]` | Ordered list of nodes. Each `NodeDescription` holds a name, a `params_type` union tag, and a typed params table. |
| `start_node` | `string`            | Name of the first node to execute each turn.                                                                     |

Wiring is encoded in the typed params table of **each source node**, not in a separate list. Every declared exit has a corresponding `<exit_name>_exit` string field (e.g. `default_exit`, `triggered_exit`). An empty string means *route to END*; any non-empty value must name another node in `nodes`.

The server validates all exit targets when processing `CreateAgentRequest`. A non-empty exit field that names a missing node is rejected with error [`3003 GraphCompilationFailed`](https://docs.tryllengine.com/v0.9/reference/error-codes.html#3xxx-agent); the message names the source node, the exit, and the missing target.

## Versioning

The current wire-protocol version is **28**. Version 28 adds `adaptation` (`ModelAdaptation`) to `GenerateParams`, `ToolCallParams`, `ClassifyIntentLLMParams`, `ClassifyAttributesLLMParams`, `GenerateAndSpeakParams` and `SummarizeParams`: a list of LoRA adapters (`loras`, each a storage-relative GGUF path and a scale) applied to the node's model (see [Use a LoRA Adapter](https://docs.tryllengine.com/v0.9/how-to/use-a-lora-adapter.html)). A version 27 server would ignore it and run the base model, so the versions must match. Version 27 moves the voice from the TTS nodes to the agent: `SpeakParams` and `GenerateAndSpeakParams` lose `tts_model_name`, `speaker_id`, `speed`, `tts_lang`, `tts_voice`, `tts_voice_description`, `tts_delivery_instruction` and (Generate and Speak) `emit_visemes`, which shifts the later fields' vtable slots. `CreateAgentRequest` gains `voice` (`VoiceProfile`), `SetAgentVoiceRequest` (95) and `SetAgentDeliveryModeRequest` (96) are appended to `MessageBody`, `TtsCapabilitiesInfo` gains `expressiveness` and `num_speakers`, and the audio format is announced once per agent turn. See [Agent voice](#agent-voice). Version 26 appends `ClassifyAttributesLLMParams` to the `NodeParams` union after `SummarizeParams` (a v25 server cannot construct that node). Version 25 adds `first_chunk_max_chars` to `GenerateAndSpeakParams` and `SpeakParams` (a v24 server would silently ignore it). Version 24 removes `CreateSessionRequest.storage_data_folder` (storage root is process-wide at startup) and adds `mode` on `ConnectionReady`. Version 23 removes `BeginUtteranceRequest.agent_id`; the server transcribes only, and the client starts any turn from the `UtteranceFinal` text. Version 22 adds `SummarizeParams` to the `NodeParams` union (after `WriteToMemoryParams`) and `PersistedSummary` to the standalone dialog-snapshot component union. Version 21 adds the viseme lip-sync track (`TtsVisemeFrame`, `GenerateAndSpeakParams.emit_visemes`; the flag moved to `VoiceProfile` in version 27). Version 20 adds experimental session-scoped mutable knowledge bases: `EnsureMutableKb` / enumerate / write / delete / clear / query / get / list / stats, per-store `ExportMutableKb` / `ImportMutableKb`, and `SaveGameMemory` / `LoadGameMemory` (unions 64–89). `LoadGameMemory.embed_timing` must be `Now` or `Lazy`; `Unspecified` is rejected (`8107`). Version 20 also adds dialog file `SaveDialog` / `LoadDialog` (90–93) with a caller `save_id`. Version 19 reordered `ClassifyIntentLLMParams` fields, a positional FlatBuffers layout break. Version 18 added `HotwordBias` to model-placement hints and changed the omitted `vad_min_silence_ms` default from 500 to 1000. Version 17 added `GetMemoryConsumptionRequest` / `GetMemoryConsumptionResponse` (union 62/63) and the `MemoryTier` / `MemoryOwnerKind` / `InferenceConsumerClass` enums. The request is session-scoped (`SessionNotReady` before `CreateSession`) but the snapshot is **server-wide** — every session on the process, the same collector as the monitor's `GET /api/memory`. It never returns `AgentBusy`. See [Inspect Server Memory](https://docs.tryllengine.com/v0.9/how-to/inspect-server-memory.html). Version 16 changed `notify_client` on the classify nodes from a bool to the `NotifyClient` enum. Version 15 added the TTS voice controls `tts_voice_description` / `tts_delivery_instruction` on `GenerateAndSpeak` and `Speak`, `ModelStatus.Incomplete`, and `ModelInfo.dependencies` / `tts_capabilities`. Version 14 added dialog export/import (unions 58–61). Version 13 replaced the four per-role engine fields on `CreateSessionRequest` with the optional `model_config` hint and added per-model memory placement. Version 12 added the session locale (`CreateSessionRequest.locale`, `SetSessionLocaleRequest`, `ModelInfo.languages`) and pause deadlines. Version 11 added the manual inference throttle — `SetInferenceThrottle` (56) — and `CreateAgentRequest.workload` (see [Inference throttle and agent workload](#inference-throttle-and-agent-workload)). Both are additive, but every client compares versions with `!=`, so any bump is a hard break at `ConnectionReady`. Version 10 was required because `RetrieveParams` gained `retrieval_mode` and `rrf_k` in the middle of the shipped FlatBuffers table (before `source` / `filter` / exits). Vtable slots are positional — mixed v9/v10 peers would decode those fields into the wrong slots — so the handshake hard-rejects a version mismatch. Version 9 replaced `ToolCallParams.notify_client`, `pause_after_tool_call`, and `history_policy` with a single `disposition: ToolCallDisposition` enum; `tool_call` NodeEvents now carry a `disposition` kv (not `history_policy`). Version 8 extended `ResumeAgentRequest` with optional `tool_results: [ToolResult {call_id, result}]` for paused `AwaitResult` tool-call batches (see [Pausing and resuming a turn](#pausing-and-resuming-a-turn)). Version 7 added dialog history mutation: `AppendInteractionsRequest` / `AppendInteractionsResponse` (52/53) and `RemoveInteractionsFromEndRequest` / `RemoveInteractionsFromEndResponse` (54/55). Version 6 added agent KV-cache lifecycle initialization and the dedicated prefill, eviction, and status request/response pairs. Version 5 added [Variables](https://docs.tryllengine.com/v0.9/concepts/agent-variables.html) messages (`UpdateAgentVariablesRequest`, `GetAgentVariablesRequest`, `GetAgentVariablesResponse`) and the trailing `CreateAgentRequest.variables` field; the version was bumped so a v5 client fails an incompatible v4 server at the `ConnectionReady` handshake rather than on the first variable request. Version 4 split the TCP connection from the logical session: `ConnectionReady` is now a connection-level hello with **no** `session_id`, and `CreateSession` (renamed from `ConfigureSession`) is mandatory and one-shot — it returns the server-allocated `session_id` in `CreateSessionResponse`. Every request before `CreateSession` is rejected with `2002 SessionNotReady`; a second `CreateSession` with `2003 SessionAlreadyExists`. (Version 2 had removed the `routes: [ExitRoute]` field from `GraphDescription` in favor of typed-param wiring; version 3 reinterpreted string-storage create/destroy as pin/unpin.)

The server does not check the client's version and never sends `5004`. Each client compares `ConnectionReady.protocol_version` with its own version and, if they differ, fails the connection locally with `5004` `ProtocolVersionMismatch`. A new client should do the same: surface both version numbers and stop reconnecting.

## Writing a new client: checklist

1. Open a TCP socket to the configured host/port.
1. Read exactly 4 bytes; interpret as little-endian uint32 → `length`.
1. Read exactly `length` bytes; decode as `Message` per `messages.fbs`.
1. Dispatch on the `body` union tag.
1. Read `ConnectionReady` first (unsolicited) and stop if its `protocol_version` differs from yours. Then send `CreateSessionRequest`; block until `CreateSessionResponse` arrives.
1. Serialise per-agent `SendMessageRequest`s client-side; do not issue a second turn before `TurnComplete`.
1. Treat any frame > 1 MiB as a fatal protocol error.
1. Tolerate unsolicited frames (e.g. `NodeEvent`) interleaved with expected responses.

## Related

- [Error codes](https://docs.tryllengine.com/v0.9/reference/error-codes.html) — numeric catalog.
- [Concept: Agents and sessions](https://docs.tryllengine.com/v0.9/concepts/agents-and-sessions.html) — state-machine view of the lifecycle above.
- `tryll/schema/messages.fbs` — canonical schema.
