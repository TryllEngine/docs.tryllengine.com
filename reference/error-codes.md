# Error Codes

Every Tryll server response that is not a normal success reply carries an `ErrorResponse` frame with a numeric **code** and a human-readable **message**. Codes are grouped by the layer that produces them, and each range has a characteristic recovery strategy.

This page lists every code a correctly-behaving server can emit. If a client ever receives a code not listed here, treat it as a protocol violation (server/client version mismatch) and log both the code and the message verbatim.

## Error-code ranges

| Range    | Layer                   | Typical recovery                                                                      |
| -------- | ----------------------- | ------------------------------------------------------------------------------------- |
| **1xxx** | Connection              | Reconnect after backoff; session state is gone.                                       |
| **2xxx** | Session                 | Re-send `CreateSessionRequest`; stop issuing requests if the server is shutting down. |
| **3xxx** | Agent                   | Recreate the agent or fix the graph; previous agent is no longer usable.              |
| **4xxx** | Inference / workflow    | Turn-local; other turns and other agents are unaffected.                              |
| **5xxx** | Protocol / framing      | Client bug or version mismatch; stop and investigate.                                 |
| **6xxx** | Model download          | User-actionable (disk, network, catalog); retry once resolved.                        |
| **7xxx** | String storage          | Fix the input and re-create the storage.                                              |
| **8xxx** | Embedded string storage | Fix the config or inline input and re-create the storage.                             |

The authoritative source of every code is `tryll/common/include/tryll/ErrorCodes.h`. Client libraries expose the same values: `Tryll::Client::ErrorCode` (C++), `tryll_client.ErrorCode` (Python), `ETryllErrorCode` (Unreal).

______________________________________________________________________

## 1xxx — Connection

These errors surface **client-side** only: the client failed to reach the server, lost the TCP connection, or timed out waiting for a reply. The server never emits 1xxx codes itself.

| Code   | Name                | Meaning                                                                                                                                     |
| ------ | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `1001` | `ConnectionFailed`  | TCP connect to the configured host/port failed outright. The server process is not running or the port is blocked.                          |
| `1002` | `ConnectionDropped` | The established connection was closed mid-session (peer close, socket error, network partition). Session state on the server is gone.       |
| `1003` | `Timeout`           | The client gave up waiting for a response frame. The socket itself may still be healthy; typically the server is overloaded or a node hung. |

**Recovery:** reconnect after exponential backoff; re-configure the session; recreate agents.

______________________________________________________________________

## 2xxx — Session

The server rejected a request because the session is not in the right state to accept it, or because the server is shutting down.

| Code   | Name                   | Meaning                                                                                                                                                |
| ------ | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `2001` | `ServerShuttingDown`   | The server is in graceful shutdown and no longer accepts new requests on this session. In-flight turns continue until they complete or time out.       |
| `2002` | `SessionNotReady`      | The client issued a request (agent, model, storage, or voice) before sending `CreateSessionRequest`. `CreateSession` is mandatory and must come first. |
| `2003` | `SessionAlreadyExists` | The client sent a second `CreateSessionRequest` on a connection that already has a session. `CreateSession` is one-shot.                               |

**Recovery:**

- `ServerShuttingDown` — stop sending; close the socket; back off before attempting to reconnect.
- `SessionNotReady` — send `CreateSessionRequest` first, then retry the original request.
- `SessionAlreadyExists` — the session is already created; do not call `CreateSession` again. Reconnect if you need a fresh session with different engine settings.

______________________________________________________________________

## 3xxx — Agent

The request targeted an agent that does not exist, is already gone, or cannot be created because its [graph](https://docs.tryllengine.com/reference/glossary.html#graph) is invalid.

| Code   | Name                     | Meaning                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ------ | ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `3001` | `InvalidAgentId`         | The `agent_id` in the request does not match any agent the server knows about. Typically a client-side bookkeeping bug.                                                                                                                                                                                                                                                                                                       |
| `3002` | `AgentAlreadyDestroyed`  | The agent was previously destroyed; its id is no longer valid. Any in-flight turn for that agent was cancelled.                                                                                                                                                                                                                                                                                                               |
| `3003` | `GraphCompilationFailed` | `CreateAgentRequest` carried a graph the server could not compile into an executable plan. The `message` field names the first validation failure (invalid start node, duplicate node names, missing model name, a malformed `grammar` on a `Generate` / `GenerateAndSpeak` node, etc.).                                                                                                                                      |
| `3004` | `AgentBusy`              | `SendMessageRequest` arrived while the agent was still processing the previous turn. Wait for `TurnComplete` before sending again. Also returned by `ChangeAgentParam` when a turn is running, and by KV-cache operations that conflict with a running/paused turn or another cache operation.                                                                                                                                |
| `3005` | `UnknownNode`            | `ChangeAgentParam` named a node that does not exist in the agent's compiled graph.                                                                                                                                                                                                                                                                                                                                            |
| `3006` | `ParamNotMutable`        | `ChangeAgentParam` targeted a parameter that cannot be changed at runtime (e.g., structural exit fields are fixed at agent creation).                                                                                                                                                                                                                                                                                         |
| `3007` | `InvalidParamValue`      | `ChangeAgentParam` supplied a value that failed the parameter's own validation (e.g., out-of-range sampling temperature, or a malformed GBNF `grammar`). The `message` field describes the constraint.                                                                                                                                                                                                                        |
| `3008` | `InvalidExitTarget`      | A node's exit field names a target node that does not exist in the graph. The `message` field identifies the offending node, exit name, and bad target string. Returned by `CreateAgentRequest`.                                                                                                                                                                                                                              |
| `3009` | `StorageOutsideRoot`     | A node's `string_storage` or `embedded_string_storage` parameter is an absolute path, or a relative path that escapes the session storage root via `..` traversal. Returned by `CreateAgentRequest`.                                                                                                                                                                                                                          |
| `3010` | `StorageFileNotFound`    | A node's storage path resolves inside the storage root but the file does not exist on disk at agent-creation time. Returned by `CreateAgentRequest`.                                                                                                                                                                                                                                                                          |
| `3011` | `StorageKindMismatch`    | A node that requires Map-kind storage (e.g. `IntentToInstruction`) was given a `.txt` (List) or Multimap file. Returned by `CreateAgentRequest`.                                                                                                                                                                                                                                                                              |
| `3012` | `AgentNotPaused`         | `ResumeAgentRequest` arrived while the agent was idle or actively running (not paused). See [Pause and resume a turn](https://docs.tryllengine.com/how-to/pause-and-resume-turns.html).                                                                                                                                                                                                                                       |
| `3013` | `UnknownVariable`        | An `UpdateAgentVariablesRequest` write targeted a [variable](https://docs.tryllengine.com/concepts/agent-variables.html) name not declared in `CreateAgentRequest.variables`. (An undeclared name in a template/filter fails earlier as `3003 GraphCompilationFailed`.)                                                                                                                                                       |
| `3014` | `VariableTypeMismatch`   | An `UpdateAgentVariablesRequest` `Assign`/`AddToSet`/`RemoveFromSet` value's type did not match the variable's declared type.                                                                                                                                                                                                                                                                                                 |
| `3015` | `InvalidVariableValue`   | An invalid variable declaration (`CreateAgentRequest.variables`: bad name grammar, duplicate name, or an over-limit name/string/set), an unknown variable op, or a set that would exceed its 4096-element cap.                                                                                                                                                                                                                |
| `3016` | `InvalidToolResults`     | `ResumeAgentRequest.tool_results` is not a complete, unique batch for the agent's currently paused [`AwaitResult`](https://docs.tryllengine.com/reference/nodes/tool-call.html) tool calls: an unknown or already-completed `call_id`, a batch sent on a non-`AwaitResult` pause, or a batch exceeding the size limits (64 entries; 128 bytes/`call_id`; 256 KiB/result; 512 KiB aggregate). The turn stays paused for retry. |

**Recovery:**

- `3001` / `3002` — recreate the agent with `CreateAgentRequest`; do not retry the original request against the bad id.
- `3003` — fix the graph and re-create. The `message` field points at the first node-level failure.
- `3004` — wait for `TurnComplete`; the Tryll client libraries already serialise per-agent sends.
- `3005` — check the node name spelling; it must match a name declared in the original `GraphDescription`.
- `3006` — remove the attempt to set that parameter; structural fields (exit routes, node type) cannot change after agent creation.
- `3007` — correct the value against the constraints described in `message` and re-send the `ChangeAgentParam` request.
- `3008` — fix the exit field on the offending node's params so it names an existing node (or leave it empty for END), then re-create the agent.
- `3009` — use a relative path that stays within the storage folder; never pass an absolute path or `..` components.
- `3010` — verify the file exists under the session's `storage_data_folder` (or the server's `storage_root`) at the path specified.
- `3011` — use the correct file type for the node: `IntentToInstruction` requires a `.json` Map file, not a `.txt` list.
- `3016` — resend `ResumeAgentRequest.tool_results` with exactly one result per pending `call_id` (from the `tool_call` event), within the size limits; the turn is still paused, so retry is safe. See [Pause and resume a turn](https://docs.tryllengine.com/how-to/pause-and-resume-turns.html).

______________________________________________________________________

## 4xxx — Inference / workflow

Errors raised by [nodes](https://docs.tryllengine.com/reference/glossary.html#node) during a turn. These end the current [turn](https://docs.tryllengine.com/reference/glossary.html#turn) with `TurnStatus::Error`; the [agent](https://docs.tryllengine.com/reference/glossary.html#agent) remains usable and can accept the next turn.

| Code   | Name                           | Meaning                                                                                                                                                                                                                                                        |
| ------ | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `4001` | `InferenceFailed`              | A [Generate](https://docs.tryllengine.com/reference/nodes/generate.html) or other inference node raised a non-recoverable runtime error (tokenizer failure, backend crash, model unloaded unexpectedly).                                                       |
| `4002` | `InferenceTimeout`             | A node exceeded its per-node time budget. Most often a symptom of a too-large prompt on a slow [inference engine](https://docs.tryllengine.com/reference/glossary.html#inference-engine).                                                                      |
| `4003` | `MaxStepsExceeded`             | The [workflow](https://docs.tryllengine.com/reference/glossary.html#workflow) exceeded the maximum number of node transitions for one turn. Usually indicates an unterminated loop in the [graph](https://docs.tryllengine.com/reference/glossary.html#graph). |
| `4004` | `KvCacheOperationFailed`       | Explicit KV-cache prefill, or agent creation with `kv_cache_initialization = Prefill`, failed while allocating, formatting, or decoding a reusable prefix.                                                                                                     |
| `4005` | `KvCacheManagementUnsupported` | The selected inference backend does not support the requested KV-cache lifecycle operation.                                                                                                                                                                    |
| `4100` | `SttModelLoadFailed`           | A `CreateVoiceInputRequest` referenced an STT (or VAD) model that could not be loaded — not on disk, or the STT engine failed to initialise it.                                                                                                                |
| `4101` | `AudioFormatUnsupported`       | Audio pushed to an open utterance is not in a format the STT engine accepts (sample rate / channel / encoding).                                                                                                                                                |
| `4102` | `VoiceInputNotFound`           | A voice-input request targeted a handle that does not exist (never created, or already destroyed).                                                                                                                                                             |
| `4103` | `UtteranceInProgress`          | `BeginUtterance` was called while an utterance is already open on that handle.                                                                                                                                                                                 |
| `4104` | `NoActiveUtterance`            | `EndUtterance` / `CancelUtterance` / audio push arrived with no utterance open.                                                                                                                                                                                |
| `4105` | `UtteranceTimeout`             | An open utterance exceeded its maximum duration and was closed by the server.                                                                                                                                                                                  |

**Recovery:** the turn has already ended; surface the error to the user or retry the user's message. For `4004` / `4005`, the failed request is a dedicated KV-cache operation rather than a turn: keep the agent, inspect the message, and retry only after resolving the underlying resource or backend issue.

______________________________________________________________________

## 5xxx — Protocol / framing

The server could not make sense of a frame the client sent. 5xxx always indicates a **client bug** or a version mismatch; there is no automated recovery.

| Code   | Name                       | Meaning                                                                                                                              |
| ------ | -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `5001` | `MalformedMessage`         | The frame could not be decoded against the FlatBuffers schema.                                                                       |
| `5002` | `UnknownMessageType`       | The frame's message-type tag is not in the server's known set.                                                                       |
| `5003` | `FrameTooLarge`            | The frame exceeds the negotiated 1 MiB maximum.                                                                                      |
| `5004` | `ProtocolVersionMismatch`  | The client speaks a wire-protocol version the server cannot parse.                                                                   |
| `5005` | `DebugCommandsUnavailable` | A `DebugCommandRequest` arrived but the debug-command channel is not compiled into this build (Production) or is disabled in config. |
| `5006` | `UnknownDebugCommand`      | A `DebugCommandRequest.command` is not a recognised verb.                                                                            |

**Recovery:** none — log, escalate, and investigate. See the [wire protocol reference](https://docs.tryllengine.com/reference/wire-protocol.html) for the framing contract.

______________________________________________________________________

## 6xxx — Model download

The server was unable to download a model. All 6xxx codes are **user-actionable**: fix the underlying condition (network, disk, catalog) and retry.

| Code   | Name                    | Meaning                                                                                                                                                                                                                 |
| ------ | ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `6001` | `DownloadFailed`        | The download transport failed (HTTP error, interrupted transfer, checksum mismatch).                                                                                                                                    |
| `6002` | `DownloadNotAvailable`  | The named model has no HuggingFace repo/files configured, so there is nothing to download (e.g. a `local_path`-only entry).                                                                                             |
| `6003` | `InsufficientDiskSpace` | The target download directory lacks free space for the model.                                                                                                                                                           |
| `6004` | `DownloadAlreadyActive` | A download for this model is already in progress on the server.                                                                                                                                                         |
| `6005` | `ModelResolutionFailed` | The named model is not in the catalog, or the engine it needs is not registered. Also returned by `LoadModel`/`CreateAgent` when a referenced model cannot be resolved (e.g. `LoadModel` for a VAD-only catalog entry). |

Note

`6006` (`ModelAutoDownloadFailed`) is retired along with the removed auto-model-downloading feature and will not be reused. A missing model referenced by `CreateAgent` / `CreateEmbeddedStringStorage` / `CreateVoiceInput` now surfaces as `GraphCompilationFailed` (3003), `ModelResolutionFailed` (6005), or `SttModelLoadFailed` (4100) instead.

**Recovery:**

- `6001` — retry after checking connectivity.
- `6002` — ask the server admin to add a matching variant to the catalog (see [Use Your Own Local Model](https://docs.tryllengine.com/how-to/use-your-own-local-model.html)), or request a different model.
- `6003` — free disk space or reconfigure `models_download_dir`.
- `6004` — wait for the in-flight download's completion frame instead of re-issuing.

______________________________________________________________________

## 7xxx — String storage

Errors raised by `CreateStringStorageRequest` and by the session storage manager at registration time. The session is unchanged on failure; the storage is not created.

| Code   | Name                         | Meaning                                                                                                                                                                                                                                                                                                                                            |
| ------ | ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `7001` | `InvalidStringStorageName`   | The storage name is empty, malformed, or reserved.                                                                                                                                                                                                                                                                                                 |
| `7002` | `StringStorageAlreadyExists` | A storage with this name is already registered for the session.                                                                                                                                                                                                                                                                                    |
| `7003` | `InvalidStringStorageData`   | The content was rejected: unreadable file, bad format, empty array.                                                                                                                                                                                                                                                                                |
| `7004` | `StorageNameInUse`           | The virtual name passed to an inline (`strings`-based) `CreateStringStorageRequest` or `CreateEmbeddedStringStorageRequest` resolves to an existing real file under the storage root. The name is rejected to prevent the virtual entry from silently shadowing the file. Use a name that does not match any filename under `storage_data_folder`. |

**Recovery:** fix the name or content and re-send the request. For `7004`, choose a virtual name that does not correspond to any file in the configured storage folder. See the [string storage reference](https://docs.tryllengine.com/reference/string-storage.html).

______________________________________________________________________

## 8xxx — Embedded string storage

Errors raised by `CreateEmbeddedStringStorageRequest`. The session is unchanged on failure; the storage is not created.

| Code   | Name                           | Meaning                                                                                                                                                                |
| ------ | ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `8001` | `InvalidEmbeddedStorageName`   | The storage name is empty, malformed, or reserved.                                                                                                                     |
| `8002` | `EmbeddedStorageAlreadyExists` | A storage with this name is already registered for the session.                                                                                                        |
| `8003` | `EmbeddedStorageBuildFailed`   | The index could not be built: missing `*.kb.json`, missing or corrupt `*.usearch`, empty inline array, or an embedding model that is not in the catalog / not on disk. |

**Recovery:** fix the config or inline input and re-send the request. See the [embedded string storage reference](https://docs.tryllengine.com/reference/embedded-string-storage.html).

______________________________________________________________________

## Client-side handling

Every Tryll client library delivers these codes through its error channel:

- **C++** — `Tryll::Client::ErrorCode` plus an `std::string` message on the `on_error` callback.
- **Python** — raised as `tryll_client.TryllError` with `.code` and `.message` attributes, or delivered to the async callback.
- **Unity** — `TryllError` value struct with `Code` (int) and `Message` (string) fields; `IsOk` is `true` when `Code == 0`. All `…Async` methods return `Task<(T result, TryllError error)>`; callers must check `error.IsOk` before using the result.
- **Unreal** — `FTryllError` USTRUCT with `Code` and `Message` properties; the `ETryllErrorCode` UENUM matches this table.

## Related

- [Wire Protocol](https://docs.tryllengine.com/reference/wire-protocol.html) — how `ErrorResponse` is framed.
- [Concept: Agents and sessions](https://docs.tryllengine.com/concepts/agents-and-sessions.html) — why 1xxx / 2xxx / 3xxx split along session lifecycle.
- [Glossary](https://docs.tryllengine.com/reference/glossary.html)
