# Python Client API Reference

Full API reference for the `tryll_client` Python package, auto-generated from Google-style docstrings. The package lives at `tryll/clients/python/tryll_client/`.

The public surface — the `__all__` re-exported from `tryll_client.__init__` — is: `TryllClient`, `ConnectedSession`, `AgentProxy`, `AgentVariables`, `VariableDecl`, `TryllError`, `WavWriter`, `GraphDescription`, `InferenceEngine`, `ManagedServer`, `ModelInfo`, `ModelStatus`, `NodeType`, the memory-snapshot types `MemorySnapshot`, `MemoryTier`, `MemoryOwnerKind`, `InferenceConsumerClass`, `MemoryDevice`, and `MemoryComponent`, and the KV-cache lifecycle types `AgentKvCacheInitialization`, `AgentKvCacheResidency`, `AgentKvCachePrefixStatus`, `AgentKvCacheStatus`, `AgentKvCachePrefillResult`, and `AgentKvCacheEvictResult`. Node-param types (`GenerateParams`, `SamplingOverrides`, `SendAnswer`, …) are imported directly from `tryll_client.graph` / `tryll_client._generated.node_params`, not from the top-level package. `ConnectedSession` is the recommended entry point for local-server deployments; obtain it via `TryllClient.run_and_connect`. Internal modules (`wire`, `codec`, `_generated`) are excluded.

______________________________________________________________________

## tryll_client

## tryll_client

tryll_client — Python client library for the Tryll server.

Quick start::

```
from tryll_client import TryllClient, InferenceEngine, GraphDescription
from tryll_client.graph import GenerateParams, SamplingOverrides
from tryll_client._generated.node_params import SendAnswer

client = TryllClient.connect("127.0.0.1", 9100)
client.create_session(InferenceEngine.LlamaCpp)

graph = (GraphDescription()
    .add_generate("generate", GenerateParams(
        system_prompt="You are a helpful assistant.",
        send=SendAnswer.Streamed,
        sampling=SamplingOverrides(temperature=0.0, seed=42),
        # default_exit defaults to "" = END
    ))
    .set_start_node("generate")
    .set_default_model_name("Llama 3.2 3B Instruct (Q4_K_M)"))

agent = client.create_agent(graph)
response = agent.send_message("Hello!")
agent.destroy()
client.shutdown()
```

Prerequisites

The CMake build for server/ must have run at least once to generate FlatBuffers Python code into tryll_client/\_generated/. Run: cmake --build server/build

### AgentProxy

```
AgentProxy(client, agent_id, node_baselines=None, variables=None)
```

Handle for one server-side agent created via `TryllClient.create_agent()`.

The proxy exposes the user-facing turn API — :meth:`send_message` and :meth:`destroy` — and caches diagnostics from the last completed turn on `last_*` properties.

Bind the proxy to its owning client and server-side agent id.

Parameters:

| Name             | Type                           | Description                                                                               | Default                                                                                                                                                            |
| ---------------- | ------------------------------ | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `client`         | `'TryllClient'`                | Parent :class:TryllClient used to send and receive wire messages on behalf of this agent. | *required*                                                                                                                                                         |
| `agent_id`       | `int`                          | Server-assigned agent identifier returned in the CreateAgentResponse.                     | *required*                                                                                                                                                         |
| `node_baselines` | \`dict[str, 'NodeParamsBase']  | None\`                                                                                    | Deep-copied params per graph node name, captured at create_agent time. Used by :meth:change_param to apply single-field mutations without caller-side bookkeeping. |
| `variables`      | \`dict\[str, tuple[int, Any]\] | None\`                                                                                    | {name: (value_type, initial_value)} declared at create_agent time, used to seed :attr:variables.                                                                   |

#### agent_id

```
agent_id
```

Server-assigned agent identifier for this proxy.

#### last_debug_info

```
last_debug_info
```

JSON diagnostics string from the most recent `send_message` call.

Returns:

| Type  | Description                                                   |
| ----- | ------------------------------------------------------------- |
| `str` | Server-produced JSON document attached to TurnComplete        |
| `str` | when diagnostics were enabled on agent creation; otherwise an |
| `str` | empty string. Empty string also before the first turn.        |

#### last_ttft_s

```
last_ttft_s
```

Time-to-first-token in seconds for the most recent turn.

Returns:

| Type    | Description |
| ------- | ----------- |
| \`float | None\`      |
| \`float | None\`      |
| \`float | None\`      |

#### last_answer_chunk_count

```
last_answer_chunk_count
```

Number of `AnswerText` chunks received for the last turn.

Typically one chunk per generated token when streaming; useful for verifying the stream was delivered incrementally.

#### last_tokens_generated

```
last_tokens_generated
```

Server-reported generated token count for the last turn.

Authoritative for both streaming and non-streaming modes. Zero if the server has not yet completed a turn.

#### paused_node

```
paused_node
```

Node this agent's turn is currently paused after; `''` when not paused.

A non-empty value means the server is holding the turn open until :meth:`resume` (or :meth:`cancel`) — until then :meth:`send_message` keeps failing with `AgentBusy` (3004).

#### set_on_answer_text

```
set_on_answer_text(cb)
```

Register (or clear) a persistent callback for streaming text chunks.

The callback is invoked on the reader thread for every `AnswerText` frame received for this agent — including frames from server-initiated turns (e.g. voice autosend after `VoiceInput.end_utterance`).

The callback signature is `(node_name: str, text: str, is_delta: bool, is_final: bool)`. `node_name` identifies which node in the workflow graph produced this text (multi-sender attribution); existing callers that only need the text can ignore it. Must return quickly and must not call blocking client methods.

Call with `None` to unregister the current callback.

Parameters:

| Name | Type                                       | Description | Default                                                                        |
| ---- | ------------------------------------------ | ----------- | ------------------------------------------------------------------------------ |
| `cb` | \`Callable\[[str, str, bool, bool], None\] | None\`      | Callable invoked with (node_name, text, is_delta, is_final), or None to clear. |

#### set_on_turn_complete

```
set_on_turn_complete(cb)
```

Register (or clear) a persistent callback for turn-complete notifications.

The callback is invoked on the reader thread once per `TurnComplete` frame for this agent. Covers both client-initiated turns (via :meth:`send_message`) and server-initiated turns (voice autosend).

The callback signature is `(status: int, debug_info: str, tokens_generated: int)` where `status` maps to the wire `TurnStatus` enum (0 = Ok, 1 = Error, 2 = Cancelled).

Call with `None` to unregister the current callback.

Parameters:

| Name | Type       | Description | Default                                                                         |
| ---- | ---------- | ----------- | ------------------------------------------------------------------------------- |
| `cb` | \`Callable | None\`      | Callable invoked with (status, debug_info, tokens_generated), or None to clear. |

#### set_on_tool_call

```
set_on_tool_call(cb)
```

Register (or clear) a callback for tool-call notifications.

The callback is invoked on the reader thread for every `NodeEvent` frame with `event_type == "tool_call"` the server sends for this agent — i.e. when the graph has a `ToolCall` node with a notifying disposition (Notify / NotifyAndAcknowledge / Pause / PauseAndAcknowledge / AwaitResult). It must return quickly and must not call any blocking :class:`TryllClient` or :class:`AgentProxy` methods.

The callback signature is `(tool_name: str, arguments_json: str)` where `arguments_json` is a compact JSON object, e.g. `'{"city": "Berlin"}'`. Parse it with :func:`json.loads` as needed.

Call with `None` to unregister the current callback.

Parameters:

| Name | Type                           | Description | Default                                                              |
| ---- | ------------------------------ | ----------- | -------------------------------------------------------------------- |
| `cb` | \`Callable\[[str, str], None\] | None\`      | Callable invoked with (tool_name, arguments_json), or None to clear. |

#### set_on_tool_call_with_id

```
set_on_tool_call_with_id(cb)
```

Register a call-ID-aware tool-call callback.

The callback receives `(call_id, tool_name, arguments_json)` on the reader thread. Echo `call_id` in a :class:`ToolResult` passed through the keyword-only `tool_results` argument of :meth:`resume_async` (preferred from this callback) or :meth:`resume` (only from a non-reader thread). The legacy :meth:`set_on_tool_call` callback remains supported.

#### set_on_error

```
set_on_error(cb)
```

Register (or clear) a per-agent client-originated error callback.

Fired on the reader thread when a user callback (tool_call, paused, etc.) raises. Does not replace server `ErrorResponse` delivery for pending requests. The error callback itself must not raise; secondary failures are swallowed to protect the reader.

#### set_on_intent_classified

```
set_on_intent_classified(cb)
```

Register (or clear) the embedding ClassifyIntent callback.

Fired on the reader thread for every `NodeEvent` with `event_type == "intent_classified"`. Call with `None` to unregister.

#### set_on_intent_llm_classified

```
set_on_intent_llm_classified(cb)
```

Register (or clear) the ClassifyIntentLLM callback.

Fired on the reader thread for every `NodeEvent` with `event_type == "intent_llm_classified"`. Call with `None` to unregister.

#### set_on_paused

```
set_on_paused(cb)
```

Register (or clear) a callback for pause notifications.

Fired on the reader thread for every `NodeEvent` with `event_type == "paused"` — emitted by the executor when it pauses the turn between nodes (see the `Pause` node and pausing `ToolCallDisposition` values). While paused, the agent still counts as busy for :meth:`send_message`, but :meth:`change_params` / :meth:`change_param` are allowed.

The callback signature is `(node_name: str, pending_exit: str)` where `pending_exit` is the exit route that will be taken on a plain (non-jump) :meth:`resume`.

Call with `None` to unregister.

#### set_on_tts_audio_format

```
set_on_tts_audio_format(cb)
```

Register (or clear) a callback for the TTS audio format frame.

Emitted by the server exactly once per turn that produces TTS audio, before the first :meth:`set_on_tts_audio` callback. The callback signature is `(node_name: str, sample_rate: int, channels: int, bits_per_sample: int)` — `node_name` is the producing node's name (multi-sender attribution; empty string if the server omitted it). Fired on the reader thread — must return quickly and must not call blocking client methods.

Call with `None` to unregister.

#### set_on_tts_audio

```
set_on_tts_audio(cb)
```

Register (or clear) a callback for streaming TTS audio chunks.

Fired on the reader thread for every `TtsAudioFrame` received for this agent. The callback signature is `(node_name: str, pcm: memoryview)` — `node_name` is the producing node's name (multi-sender attribution; matches the preceding :meth:`set_on_tts_audio_format` call for the same producer). The `pcm` argument is a zero-copy `memoryview` over int16 LE PCM at the format declared by the preceding format callback. The view is valid only during the call — copy with `bytes(pcm)` or `np.frombuffer(pcm, dtype=np.int16).copy()` if needed.

`TtsAudioFrame` carries no `is_final` flag — end-of-stream is signalled by `TurnComplete`. Wire :meth:`set_on_turn_complete` (or close your sink in the surrounding `with`-block) to flush.

Call with `None` to unregister.

#### set_on_node_event

```
set_on_node_event(cb)
```

Register (or clear) a generic `NodeEvent` fallback callback.

Fired on the reader thread for any `NodeEvent` whose `event_type` is unrecognised or whose typed callback (e.g. :meth:`set_on_tool_call`, :meth:`set_on_intent_classified`) is not registered. When a typed callback handles the event, this one is NOT invoked for that event.

The callback signature is `(node_name: str, event_type: str, kv_pairs: list[tuple[str, str]])`.

Call with `None` to unregister.

#### send_message

```
send_message(text, timeout=120.0)
```

Send a user message and return the complete assistant response.

Blocks until `TurnComplete` is received from the server, accumulating all `AnswerText` chunks into a single string. Diagnostics from `TurnComplete` (`debug_info`, `tokens_generated`) are cached on the `last_*` properties.

Parameters:

| Name      | Type    | Description                               | Default    |
| --------- | ------- | ----------------------------------------- | ---------- |
| `text`    | `str`   | User-turn text to send to the agent.      | *required* |
| `timeout` | `float` | Maximum seconds to wait for TurnComplete. | `120.0`    |

Returns:

| Type  | Description                                    |
| ----- | ---------------------------------------------- |
| `str` | The full concatenated assistant response text. |

Raises:

| Type         | Description                                                                                    |
| ------------ | ---------------------------------------------------------------------------------------------- |
| `TryllError` | On server-reported errors, decode failures, or if timeout elapses before TurnComplete arrives. |

#### send_message_async

```
send_message_async(text)
```

Send a user message without blocking; return a Future for the result.

Unlike :meth:`send_message`, this returns immediately so the caller can act mid-turn (e.g. watch streaming callbacks and call :meth:`cancel`). The Future resolves with a :class:`TurnResult` on `TurnComplete` (including a cancelled turn, where `status == TurnStatus.Cancelled`) or raises :class:`~tryll_client.errors.TryllError` on a server error.

Await it via `fut.result(timeout)` (raises `concurrent.futures.TimeoutError` if the turn does not finish in time). Streaming chunks still arrive via the `set_on_answer_text` / `set_on_tts_audio` callbacks.

Note: use :meth:`cancel` to cancel the turn — do **not** call `fut.cancel()`, which only cancels a scheduled call, not the server-side turn.

#### change_param

```
change_param(node_name, path, value, timeout=30.0)
```

Apply a single-field mutation using a dotted attribute path.

Clones the baseline params captured at `create_agent`, coerces *value* to the leaf field type (when *value* is a string), assigns via :func:`~tryll_client._generated.node_params_codec.set_dotted`, and sends the full params object. On success the baseline is updated so later mutations compose.

Example::

```
agent.change_param("generate", "sampling.temperature", "0.5")
```

Parameters:

| Name        | Type    | Description                                                            | Default    |
| ----------- | ------- | ---------------------------------------------------------------------- | ---------- |
| `node_name` | `str`   | Instance name of the target node.                                      | *required* |
| `path`      | `str`   | Dotted path to the field (e.g. "sampling.temperature").                | *required* |
| `value`     | `Any`   | New value (string from JSON/dialog scripts, or a native Python value). | *required* |
| `timeout`   | `float` | Maximum seconds to wait for the server Ack.                            | `30.0`     |

Raises:

| Type         | Description                                                              |
| ------------ | ------------------------------------------------------------------------ |
| `TryllError` | If node_name is unknown (code 3005) or the server reports another error. |

#### change_params

```
change_params(node_name, params, timeout=30.0)
```

Apply a typed node-parameter update to a workflow node at runtime.

The typed params object must match the concrete type of the target node. Structural fields must match the create-time values or the server returns `ParamNotMutable`. On success, updates the internal baseline used by :meth:`change_param`.

Parameters:

| Name        | Type               | Description                                 | Default    |
| ----------- | ------------------ | ------------------------------------------- | ---------- |
| `node_name` | `str`              | Instance name of the target node.           | *required* |
| `params`    | `'NodeParamsBase'` | Typed node parameters (generated DTO).      | *required* |
| `timeout`   | `float`            | Maximum seconds to wait for the server Ack. | `30.0`     |

Raises:

| Type         | Description                                                                                                            |
| ------------ | ---------------------------------------------------------------------------------------------------------------------- |
| `TryllError` | On server-reported errors (AgentBusy 3004, UnknownNode 3005, ParamNotMutable 3006, InvalidParamValue 3007) or timeout. |

#### destroy

```
destroy(timeout=30.0)
```

Request agent destruction on the server and wait for `Ack`.

After this call returns, the proxy is no longer usable; further `send_message` calls will raise :class:`TryllError`.

Parameters:

| Name      | Type    | Description                                   | Default |
| --------- | ------- | --------------------------------------------- | ------- |
| `timeout` | `float` | Maximum seconds to wait for the Ack response. | `30.0`  |

Raises:

| Type         | Description                           |
| ------------ | ------------------------------------- |
| `TryllError` | On server-reported errors or timeout. |

#### cancel

```
cancel(mode=0, timeout=30.0)
```

Cooperatively cancel the agent's in-flight turn and wait for `Ack`.

The active `send_message` ends with `TurnStatus.Cancelled`. Call this from a different thread than the blocking `send_message` to interrupt a turn mid-flight. No-op server-side if the agent is idle.

Parameters:

| Name      | Type    | Description                                                                                                                                                  | Default |
| --------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------- |
| `mode`    | `int`   | :class:~tryll_client.graph.CancelMode — StopAndKeep (0, default) keeps the partial interaction; StopAndDiscard (1) removes it as if the turn never happened. | `0`     |
| `timeout` | `float` | Maximum seconds to wait for the Ack response.                                                                                                                | `30.0`  |

Raises:

| Type         | Description                           |
| ------------ | ------------------------------------- |
| `TryllError` | On server-reported errors or timeout. |

#### resume

```
resume(resume_node='', timeout=30.0, *, tool_results=())
```

Resume a turn that is currently paused and wait for `Ack`.

**Not safe on the reader thread** (tool-call / paused callbacks): blocking here deadlocks the sole frame reader. Use :meth:`resume_async` from those callbacks instead.

Parameters:

| Name           | Type                   | Description                                                                                                                       | Default                                                                                                                                                                                                                                              |
| -------------- | ---------------------- | --------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `resume_node`  | `str`                  | Empty continues via the paused node's pending exit route; non-empty jumps to that node by name (must exist in the agent's graph). | `''`                                                                                                                                                                                                                                                 |
| `timeout`      | `float`                | Maximum seconds to wait for the Ack response.                                                                                     | `30.0`                                                                                                                                                                                                                                               |
| `tool_results` | \`Sequence\[ToolResult | tuple[str, str]\]\`                                                                                                               | Complete result batch for the paused tool-call pass. Each result must echo the call_id surfaced by :meth:set_on_tool_call_with_id. Typed :class:ToolResult values are preferred; (call_id, result) pairs are accepted for codec-level compatibility. |

Raises:

| Type         | Description                                                                                                                                     |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `TryllError` | AgentNotPaused (3012) if the agent is not currently paused, UnknownNode (3005) if resume_node does not name a node in the graph, or on timeout. |

#### resume_async

```
resume_async(resume_node='', *, tool_results=())
```

Resume a paused turn without blocking; returns a Future for the Ack.

Safe to call from reader-thread callbacks. The Future resolves to `None` on success or raises :class:`TryllError` on failure.

#### prefill_kv_cache

```
prefill_kv_cache(timeout=30.0)
```

Synchronise eligible language-model contexts to the reusable prefix.

#### evict_kv_cache

```
evict_kv_cache(timeout=30.0)
```

Release eligible raw contexts while retaining agent state.

#### get_kv_cache_status

```
get_kv_cache_status(timeout=30.0)
```

Return aggregate residency and reusable-prefix state.

#### append_interactions

```
append_interactions(interactions, timeout=30.0)
```

Append scripted user/assistant interactions without running the graph.

Each item is `(user_message, assistant_message)`. Empty / `None` strings omit that side; both empty → skipped. Idle-only (`AgentBusy` 3004 while running/paused).

Returns:

| Type  | Description                               |
| ----- | ----------------------------------------- |
| `int` | Number of interactions actually appended. |

#### remove_interactions_from_end

```
remove_interactions_from_end(count, timeout=30.0)
```

Remove whole interactions from the end of the dialog.

`count == 0` is a no-op; `count >= size` clears all. Idle-only.

Returns:

| Type  | Description                              |
| ----- | ---------------------------------------- |
| `int` | Number of interactions actually removed. |

Raises:

| Type         | Description                                            |
| ------------ | ------------------------------------------------------ |
| `ValueError` | If count is negative.                                  |
| `TryllError` | On server-reported errors (AgentBusy 3004) or timeout. |

#### export_dialog

```
export_dialog(timeout=30.0)
```

Return the agent's completed dialog as an owned snapshot. Idle-only.

#### import_dialog

```
import_dialog(snapshot, compatibility=0, timeout=30.0)
```

Replace the agent's dialog with *snapshot*. Idle-only. Never appends.

### ToolResult

```
ToolResult(call_id, result)
```

One model-visible result correlated to a tool-call `call_id`.

### WavWriter

```
WavWriter(output_path)
```

Write streaming TTS audio to a 16-bit mono WAV file.

Lazily opens the output file on the first format callback, appends every TTS audio chunk, and closes the file on :meth:`on_turn_complete` (or explicit :meth:`close`).

`TtsAudioFrame` does not carry an `is_final` flag — end-of-stream is signalled by `TurnComplete`. Use :meth:`bind_to` (which wires all three callbacks) or register :meth:`on_turn_complete` yourself.

**Single producer per instance.** A `WavWriter` captures exactly one TTS producer per turn — a second :meth:`on_format` call before :meth:`close` raises :class:`RuntimeError` instead of silently reopening (and truncating) the file. This is not a limitation in practice: the voice loop recreates a fresh writer for every turn, and GenerateAndSpeak/Speak each emit at most one producer per turn. A graph with two TTS-emitting nodes active in the same turn needs one `WavWriter` per node (keyed by `node_name`), not one shared instance.

Bind a sink path; the file is created lazily on the format callback.

#### on_format

```
on_format(node_name, sample_rate, channels, bits_per_sample)
```

Open the output file using the announced format.

Raises :class:`RuntimeError` if called a second time before :meth:`close` — multi-producer capture into a single `WavWriter` is not supported (see the class docstring); allowing it would silently truncate the first producer's audio.

#### on_audio

```
on_audio(node_name, pcm)
```

Append one TTS chunk. Silently drops chunks that arrive before format.

`node_name` is accepted for signature compatibility (see :meth:`on_format`) but unused.

#### on_turn_complete

```
on_turn_complete(status=0, debug_info='', tokens_generated=0)
```

Close the file. Signature matches :meth:`AgentProxy.set_on_turn_complete`.

#### close

```
close()
```

Close the underlying file. Idempotent.

#### bind_to

```
bind_to(agent, output_path)
```

Create a :class:`WavWriter` and wire it to *agent*'s TTS callbacks.

Registers :meth:`on_format`, :meth:`on_audio`, and :meth:`on_turn_complete` on *agent*. Note that this overrides any previously registered `turn_complete` callback on the agent — wire the three handlers manually if you need a custom turn-complete sink.

### ConnectedSession

```
ConnectedSession(*, server, client)
```

RAII wrapper that owns a :class:`~tryll_client.managed_server.ManagedServer` and a connected :class:`TryllClient`.

Obtain via :meth:`TryllClient.run_and_connect`. Use as a context manager (recommended) to guarantee teardown in the right order — client first, then server::

```
with TryllClient.run_and_connect(exe=Path("..."), port=9100) as session:
    session.client.create_session([(ModelRole.Language, InferenceEngine.LlamaCpp)])
    agent = session.client.create_agent(graph)
    print(agent.send_message("hi"))
```

#### client

```
client
```

The connected :class:`TryllClient`.

#### server

```
server
```

The running :class:`~tryll_client.managed_server.ManagedServer`.

#### shutdown

```
shutdown()
```

Shut the client down, then stop the server. Idempotent.

### LoadModelResult

```
LoadModelResult(placement, backend)
```

What a :meth:`TryllClient.load_model` call actually got.

Both fields describe the load that happened, not the load that was asked for. `placement` is `Ram` when a `Vram` request could not be honoured (no GPU backend, or a CPU-only Sherpa ONNX Runtime build), and it is the *resident* value when another session already loaded the model the other way — placement is fixed at load time and deliberately is not part of the model cache key, so the first loader wins.

`backend` is the device actually serving the model (`"cuda"`, `"vulkan"`, `"rocm"`, `"cpu"`, or an ONNX Runtime provider), and is empty when the load resolved no backend. A client never chooses it: which GPU API is available is a host decision, process-wide and one-shot for llama.cpp and compile-time for Sherpa.

### TryllClient

```
TryllClient(sock)
```

Synchronous TCP session to the Tryll server.

Usage::

```
client = TryllClient.connect("127.0.0.1", 9100)
client.create_session([(ModelRole.Language, InferenceEngine.LlamaCpp)])
graph = GraphDescription().add_node(...).set_start_node(...)
# Exit fields (e.g. default_exit="") live on each node's typed params
agent = client.create_agent(graph)
response = agent.send_message("Hello")
agent.destroy()
client.shutdown()
```

#### connect

```
connect(host='127.0.0.1', port=9100, timeout=30.0)
```

Connect to the Tryll server and wait for ConnectionReady.

#### run_and_connect

```
run_and_connect(*, exe, host='127.0.0.1', port=9100, cwd=None, extra_args=(), stdout=None, stderr=None, start_timeout=30.0, stop_timeout=8.0, connect_timeout=30.0, idle_shutdown_timeout=60)
```

Spawn tryll_server and connect — recommended one-call factory.

Starts a local server process (passing `--port <port>` and, unless *idle_shutdown_timeout* is 0, `--idle-shutdown-timeout <N>` on the command line), waits for its TCP port to be ready, then opens a session. The returned :class:`ConnectedSession` owns both the server process and the client; use it as a context manager to guarantee clean teardown::

```
with TryllClient.run_and_connect(exe=Path("..."), port=9100) as session:
    session.client.create_session([(ModelRole.Language, InferenceEngine.LlamaCpp)])
    agent = session.client.create_agent(graph)
```

If the connected server's advertised `exe_path` does not match *exe* (a stranger raced us onto the port), the connection is torn down, the wrong process is hard-killed, and a fresh server is spawned on a new free port — up to a few attempts before raising.

Parameters:

| Name                    | Type            | Description                                                                                                                                                             | Default                                                           |
| ----------------------- | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| `exe`                   | \`Path          | str\`                                                                                                                                                                   | Path to tryll_server[.exe]. Required.                             |
| `host`                  | `str`           | Host for TCP probe and :meth:connect (default 127.0.0.1).                                                                                                               | `'127.0.0.1'`                                                     |
| `port`                  | `int`           | Port passed as --port to the server (default 9100).                                                                                                                     | `9100`                                                            |
| `cwd`                   | \`Path          | None\`                                                                                                                                                                  | Working directory for the child process (defaults to exe.parent). |
| `extra_args`            | `Sequence[str]` | Additional CLI arguments after --port <port>.                                                                                                                           | `()`                                                              |
| `stdout`                | \`Path          | None\`                                                                                                                                                                  | File path for child stdout (None = discard).                      |
| `stderr`                | \`Path          | None\`                                                                                                                                                                  | File path for child stderr (None = discard).                      |
| `start_timeout`         | `float`         | Seconds to wait for the TCP port to open (default 30).                                                                                                                  | `30.0`                                                            |
| `stop_timeout`          | `float`         | Seconds to wait for graceful server exit (default 8).                                                                                                                   | `8.0`                                                             |
| `connect_timeout`       | `float`         | Seconds to wait for :class:ConnectionReady (default 30).                                                                                                                | `30.0`                                                            |
| `idle_shutdown_timeout` | `int`           | Seconds of idle after which the launched server self-exits (default 60); see :meth:ManagedServer.start. Pass 0 only when the caller has its own explicit reap strategy. | `60`                                                              |

Returns:

| Name | Type                 | Description                                                  |
| ---- | -------------------- | ------------------------------------------------------------ |
| `A`  | `'ConnectedSession'` | class:ConnectedSession whose server is running and client is |
|      | `'ConnectedSession'` | connected.                                                   |

Raises:

| Type                | Description                                                                                 |
| ------------------- | ------------------------------------------------------------------------------------------- |
| `FileNotFoundError` | If exe does not exist.                                                                      |
| `TimeoutError`      | If the port does not open within start_timeout.                                             |
| `TryllError`        | If the session handshake fails, or identity verification keeps failing after a few retries. |

#### set_session_locale

```
set_session_locale(locale, timeout=30.0)
```

Change the session language without tearing the session down.

Same BCP-47 grammar as :meth:`create_session`'s `locale`; an empty string *clears* the locale, restoring pre-locale behaviour everywhere.

```
Applies to every agent, existing and future, from its next turn: TTS
language and the locale template keys are read per turn, not captured.
A new voice input picks up the new STT language hint; one already open
keeps its recognizer. Storage paths re-resolve on the next load, but
content already bound to a node (a Retrieve KB, canned lines) does NOT
swap - those params are structural. Use this for the voice + template
half of a language change; switching authored content still means
recreating agents.
```

Not turn-atomic: a turn already running may render its prompt under the old locale and synthesize under the new one. Send it between turns if that matters.

Parameters:

| Name      | Type    | Description                                 | Default    |
| --------- | ------- | ------------------------------------------- | ---------- |
| `locale`  | `str`   | BCP-47 tag ("de", "pt-BR"), or '' to clear. | *required* |
| `timeout` | `float` | Maximum seconds to wait for the server Ack. | `30.0`     |

Raises:

| Type         | Description                                     |
| ------------ | ----------------------------------------------- |
| `TryllError` | InvalidLocale (2004) when the tag is malformed. |

#### create_session

```
create_session(engines=None, game_name='', timeout=30.0, storage_data_folder='', locale='', placement_hints=None, hotword_hints=None)
```

Create the logical session for this connection (mandatory, one-shot).

Sends CreateSessionRequest and waits for CreateSessionResponse, which carries the server-allocated `session_id` (stored on :attr:`session_id`). Must be called exactly once after :meth:`connect` and before :meth:`create_agent` or any model/storage/voice call — those are rejected with `SessionNotReady` until the session exists. A second call is rejected by the server with `SessionAlreadyExists`.

Parameters:

| Name                  | Type                                       | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | Default                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| --------------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `engines`             | \`'Optional\[Sequence\[tuple\[ModelRole    | int, InferenceEngine                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | int\]\]\]'\`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `game_name`           | `str`                                      | Integration identifier for telemetry grouping (e.g. "qa-and-eval"). Leave empty for anonymous sessions.                                                                                                                                                                                                                                                                                                                                                                                                             | `''`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `timeout`             | `float`                                    | Maximum seconds to wait for the server response.                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | `30.0`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `placement_hints`     | \`'Optional\[Mapping\[str, MemoryPlacement | int\]\]'\`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | Optional per-model memory placement, e.g. {"Llama 3.1 8B": MemoryPlacement.Ram}. Sparse: a model you do not list means :attr:MemoryPlacement.Auto. These are hints — a host that cannot honour one downgrades it and reports what it actually did, so read the result from :meth:load_model rather than assuming. LoadModelRequest.placement overrides a hint for a single load.                                                                                                                                                                                                                                                                                                                                                   |
| `hotword_hints`       | \`'Optional\[Mapping\[str, HotwordBias     | int\]\]'\`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | Optional per-model hotword biasing, e.g. {"Parakeet TDT 0.6B v3 (int8)": HotwordBias.On}. Sparse: a model you do not list follows its catalog entry, which biases when that entry sets a non-zero stt_hotwords_score. Unlike placement_hints this is not downgradable. Biasing needs beam-search decoding, which a recognizer fixes when it is built, so this selects which recognizer the session gets — and two sessions that disagree about one model hold two of them. On costs a few percent of decode time and changes that model's transcripts whether or not you pass a lexicon; Off makes :meth:create_voice_input reject a hotwords_storage_path with HotwordBiasDisabled (4106) rather than accept it and bias nothing. |
| `storage_data_folder` | `str`                                      | Optional per-session storage data folder. Relative storage paths in node params (and voice-input hotwords) resolve against this folder. Empty (default) means the server falls back to its configured storage_root.                                                                                                                                                                                                                                                                                                 | `''`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `locale`              | `str`                                      | Optional session language as a BCP-47 tag ("de", "de-DE", "pt-BR"). One setting the whole voice + text stack derives from: the default tts_lang for every GenerateAndSpeak/Speak node, the language hint for multilingual STT models, loc/<locale>/ siblings prefix relative storage paths prefer, and the language Mustache key a prompt can write a "reply in X" directive against. Empty (default) leaves every one of those at its pre-locale behaviour. A malformed tag is rejected with InvalidLocale (2004). | `''`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |

#### set_inference_throttle

```
set_inference_throttle(level)
```

Report how hard the server should yield the GPU back to the game.

Fire-and-forget: no response, no error surfaced locally. Never sent automatically — a session that never calls this runs unthrottled, identically to a client on an older protocol version.

Parameters:

| Name    | Type    | Description                                                                                                 | Default    |
| ------- | ------- | ----------------------------------------------------------------------------------------------------------- | ---------- |
| `level` | `float` | 0 = full speed (default), 1 = maximum yielding. Clamped server-side; out-of-range values do not raise here. | *required* |

#### create_string_storage

```
create_string_storage(name, strings=None, file_path=None, timeout=10.0)
```

Create a named StringStorage on the server.

Provide either *strings* (inline list) or *file_path* (server-side file path). The storage can then be referenced by name in node params via `string_storage`.

#### create_keyed_string_storage

```
create_keyed_string_storage(name, keys, values, kind=StringStorageKind.Map, timeout=10.0)
```

Create a Map or Multimap-kind StringStorage on the server.

*keys* and *values* must be the same length. For `Map` kind all keys must be unique.

The storage can then be referenced by name in `IntentToInstructionNode` (and other keyed-consumer nodes) via the `string_storage` param.

#### destroy_string_storage

```
destroy_string_storage(name, timeout=10.0)
```

Destroy a named StringStorage on the server.

Nodes that already hold the storage are unaffected.

#### create_embedded_string_storage

```
create_embedded_string_storage(name, config_path=None, strings=None, embedding_model=None, timeout=None)
```

Create a named EmbeddedStringStorage on the server.

Path A: supply *config_path* (server-side `*.json`). Path B: supply *strings* (inline list) + *embedding_model*. Returns an EmbeddedStorageInfo with record_count and embedding_dim.

#### destroy_embedded_string_storage

```
destroy_embedded_string_storage(name, timeout=10.0)
```

Destroy a named EmbeddedStringStorage on the server.

Nodes that already hold the storage are unaffected.

#### create_voice_input

```
create_voice_input(model_name, sample_rate=16000, channels=1, bits_per_sample=16, vad_threshold=0.5, vad_min_silence_ms=1000, vad_speech_pad_ms=250, hotwords_storage_path='', hotwords_score=1.5, timeout=30.0)
```

Create a server-side VoiceInput session for speech-to-text.

Returns a :class:`~tryll_client.voice_input.VoiceInput` handle managing the server-side session. Use as a context manager (recommended) for guaranteed cleanup.

Parameters:

| Name                    | Type    | Description                                                                                                                                                                                                                                                            | Default    |
| ----------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
| `model_name`            | `str`   | Catalog name of the STT model, e.g. "Parakeet TDT 0.6B v3 (int8)".                                                                                                                                                                                                     | *required* |
| `sample_rate`           | `int`   | Sample rate of audio buffers you will push (Hz).                                                                                                                                                                                                                       | `16000`    |
| `channels`              | `int`   | Channel count (1 = mono, typical for mic capture).                                                                                                                                                                                                                     | `1`        |
| `bits_per_sample`       | `int`   | Bit depth of raw PCM samples (only 16-bit is supported).                                                                                                                                                                                                               | `16`       |
| `vad_threshold`         | `float` | Silero VAD speech-probability threshold (0.0–1.0).                                                                                                                                                                                                                     | `0.5`      |
| `vad_min_silence_ms`    | `int`   | Silence duration (ms) that closes a speech segment.                                                                                                                                                                                                                    | `1000`     |
| `vad_speech_pad_ms`     | `int`   | Padding (ms) added around detected speech.                                                                                                                                                                                                                             | `250`      |
| `hotwords_storage_path` | `str`   | Relative path (resolved against the session storage folder) to a StringStorage (kind=List) config whose phrases the STT decoder will bias toward. Loaded on demand — no prior create_string_storage needed. Pass an empty string (default) to disable hotword biasing. | `''`       |
| `hotwords_score`        | `float` | How strongly matched phrases are weighted against one another during beam search. Measured on Parakeet TDT 0.6B v3: 1.8 recovered 5 of 8 target terms, 2.5 recovered 6 of 8. A lexicon line may override this for itself by ending with :<number>.                     | `1.5`      |
| `timeout`               | `float` | Maximum seconds to wait for the server response.                                                                                                                                                                                                                       | `30.0`     |

Raises:

| Type         | Description                                                                                   |
| ------------ | --------------------------------------------------------------------------------------------- |
| `TryllError` | If the model cannot be found, the hotwords storage file is missing, or the request times out. |

#### create_agent

```
create_agent(graph, enable_diagnostics=False, timeout=None, maintain_dialogue_history=True, variables=None, kv_cache_initialization=AgentKvCacheInitialization.AllocateOnly, workload=AgentWorkload.Interactive)
```

Create a server-side agent and return a proxy handle.

Parameters:

| Name                        | Type               | Description                                                                                                                                                                                                                      | Default                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| --------------------------- | ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `graph`                     | `GraphDescription` | Fully-built graph description.                                                                                                                                                                                                   | *required*                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `enable_diagnostics`        | `bool`             | When True the server serialises per-node execution data into TurnComplete.debug_info for every turn.                                                                                                                             | `False`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `timeout`                   | \`float            | None\`                                                                                                                                                                                                                           | Maximum seconds to wait for the response. Defaults to 30 s.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `maintain_dialogue_history` | `bool`             | When True (default) the agent keeps its full dialogue history. When False the history is discarded after each turn — a stateless agent, useful for technical/classification agents that should not store or project prior turns. | `True`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `variables`                 | \`dict[str, Any]   | None\`                                                                                                                                                                                                                           | Declares this agent's per-agent variables — a {name: initial_value} mapping. Values may be consumed from Mustache templates as {{var.<name>}} and the Retrieve/ClassifyIntent filter grammar as {"var": "<name>"}. Values may instead be VariableDecl(initial_value, allow_output_substitution=True) to permit output marker substitution for that declaration. Plain values default the permission to False. The wire type is inferred from each value (bool/int/float/str/ set or list of str) and locked for the agent's lifetime; this is the complete declaration — later writes to undeclared names raise TryllError (3013). |
| `workload`                  | \`AgentWorkload    | int\`                                                                                                                                                                                                                            | :class:AgentWorkload (or its ordinal). Whether a human ever waits on this agent's turns — pass AgentWorkload.Background for world-logic/evaluator agents so the inference scheduler paces or defers their work first under GPU pressure. Purely a scheduling hint; it never changes node semantics. Fixed for the agent's lifetime — it is not a node parameter, so change_param cannot alter it; recreate the agent instead.                                                                                                                                                                                                      |

#### list_models

```
list_models(timeout=10.0)
```

Request every model in the server's catalog, each reporting its engine.

#### get_memory_consumption

```
get_memory_consumption(timeout=10.0)
```

Pull a server-wide RAM/VRAM snapshot.

Session-scoped (must follow create_session) but the numbers cover every session on the process — the same view as the server monitor. Never fails with AgentBusy.

#### load_model

```
load_model(model_name, timeout=300.0, placement=None)
```

Explicitly load and pin a model into memory.

The model stays in memory until :meth:`unload_model` is called, regardless of whether any agents are using it. Raises :class:`TryllError` if the model cannot be resolved or loaded.

Parameters:

| Name        | Type               | Description | Default |
| ----------- | ------------------ | ----------- | ------- |
| `placement` | \`'MemoryPlacement | int         | None'\` |

Returns:

| Name       | Type              | Description                                                           |
| ---------- | ----------------- | --------------------------------------------------------------------- |
|            | `LoadModelResult` | class:LoadModelResult — where the model actually ended up and which   |
|            | `LoadModelResult` | backend is serving it. Read this rather than assuming the request was |
| `honoured` | `LoadModelResult` | a Vram request on a host with no GPU backend reports                  |
|            | `LoadModelResult` | Ram, and a model another session already loaded reports its           |
|            | `LoadModelResult` | resident placement. A mismatch against an explicit request is also    |
|            | `LoadModelResult` | logged at warning level.                                              |

#### unload_model

```
unload_model(model_name, timeout=30.0)
```

Unpin a previously pinned model.

If no agents are currently using the model it is freed immediately; otherwise it will be freed when the last agent using it is destroyed.

#### download_model

```
download_model(model_name, on_progress=None, timeout=1800.0)
```

Initiate a model download on the server and block until complete.

*on_progress* is called with (model_name, bytes_downloaded, total_bytes, percent) for each progress frame received. Raises TryllError on failure.

#### send_debug_command

```
send_debug_command(command, payload='', *, await_response=True, timeout=10.0)
```

Send a dev-only debug command to the server.

The server handles this only in non-Production builds with `debug_commands.enabled = true`; otherwise it returns a TryllError.

`command` is a verb (e.g. `"crash"`); `payload` is verb-specific (for `crash` it is the mode token: `nullderef` | `abort` | `throw` | `throw_posted` | `stackoverflow` | `fastfail`; empty defaults to `nullderef`).

Set `await_response=False` for fatal verbs like `crash` that kill the server and never reply — the call sends the frame and returns immediately; the caller detects success via the ensuing TCP disconnect. When `await_response` is True, returns the server's `result` string.

#### wait_for_disconnect

```
wait_for_disconnect(timeout=10.0)
```

Block until the server drops the connection, or *timeout* elapses.

The reader thread exits when the socket reaches EOF / resets (e.g. the server process died), so joining it is a reliable disconnect signal. Returns True if the connection dropped within *timeout*. Used after a fatal debug command (`crash`) whose only success signal is the drop.

#### shutdown

```
shutdown()
```

Close the socket and stop the reader thread.

### TryllError

```
TryllError(message, code=0)
```

Bases: `Exception`

Raised on server errors, protocol errors, or timeouts.

Attributes:

| Name   | Type | Description                                                                                                                                                                                                                                                 |
| ------ | ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `code` |      | Numeric error code from the server's ErrorCodes.h ranges (1xxx connection, 2xxx session, 3xxx agent, 4xxx model, 5xxx node, 6xxx graph, 7xxx string storage, 8xxx embedded storage). Zero for client-originated errors such as timeouts or a closed socket. |

Initialise a Tryll client error.

Parameters:

| Name      | Type  | Description                                                                                                                                                       | Default    |
| --------- | ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
| `message` | `str` | Human-readable description of the failure, typically copied from the server's ErrorResponse.message field or synthesised locally (e.g. "Timeout (30s) waiting…"). | *required* |
| `code`    | `int` | Numeric error code from ErrorCodes.h ranges; zero for client-originated errors.                                                                                   | `0`        |

#### __str__

```
__str__()
```

Return the message, prefixed with `[code]` when non-zero.

### GraphDescription

```
GraphDescription()
```

Fluent builder for a workflow graph sent to the server.

Typical usage::

```
graph = (GraphDescription()
         .add_generate("gen", GenerateParams(
             model_name="qwen2.5-0.5b-instruct",
             system_prompt="You are helpful."))
         .set_start_node("gen"))
agent = await client.create_agent(graph)
```

#### add_node

```
add_node(name, params)
```

Generic typed node add; accepts any concrete NodeParamsBase subclass.

#### set_start_node

```
set_start_node(name)
```

Set the entry-point node name.

#### set_default_model_name

```
set_default_model_name(name)
```

Set the fallback model name for nodes that need one.

### InferenceEngine

Bases: `IntEnum`

Inference backend. Mirrors the FlatBuffers `InferenceEngine` enum.

### HotwordBias

Bases: `IntEnum`

Whether a session wants hotword biasing for one STT model.

Overrides the model catalog's `stt_hotwords_score`. Unlike :class:`MemoryPlacement` this is not a downgradable hint: biasing needs beam-search decoding, which a recognizer fixes when it is constructed, so the answer selects *which recognizer* the session is served. Two sessions that disagree about one model hold two recognizers between them.

### MemoryPlacement

Bases: `IntEnum`

Where a model's weights should live. Mirrors `MemoryPlacement`.

Always a *hint*: a host that cannot honour it downgrades and reports what it did, so read the resolved value back rather than assuming. Which GPU API provides VRAM (CUDA / Vulkan / HIP) is a host decision and is deliberately not expressible here.

`Auto` is all-or-nothing in this version — device memory when the host has a GPU, system memory otherwise. It does not size offload to free memory.

### ModelDependency

```
ModelDependency(id, role, status, size_bytes=0, folder='', local_path='', failure_reason='')
```

Editor-facing status of one shared-asset dependency of a parent model.

### ModelInfo

```
ModelInfo(name, status, hf_repo, size_bytes, languages=(), dependencies=(), tts_capabilities=None)
```

Summary information for one model returned by `list_models()`.

### ModelRole

Bases: `IntEnum`

What a model is for. Mirrors the FlatBuffers `ModelRole` enum.

Used to qualify an engine declaration in :meth:`TryllClient.create_session`. The role is redundant for lookup — the server reads each model's engine from its own catalog entry — and exists so a rejection can name what the mismatch actually is.

### ModelStatus

Bases: `IntEnum`

Status of a model as reported by the server.

### NodeType

Bases: `IntEnum`

Node type ordinal. Mirrors the NodeParams union order in nodes.fbs (0-based; the wire discriminator is this value + 1).

The union discriminant in `NodeParams` supersedes this for the wire format; `NodeType` is kept for client-side ergonomics and generated convenience APIs.

### TtsCapabilities

```
TtsCapabilities(voice_description=False, delivery_instruction=False, reference_voice=False, speed=False, language_per_call=False, voice_presets=(), inline_tags=())
```

Structured TTS capabilities advertised by the catalog.

### TtsInlineTag

```
TtsInlineTag(token, kind='', verification='')
```

Model-native inline control token declared for authoring.

### TtsVoicePreset

```
TtsVoicePreset(id, label, description='')
```

Catalog-advertised speaker/description preset.

### ManagedServer

```
ManagedServer(*, exe, host='127.0.0.1', port=9100, cwd=None, extra_args=(), stdout=None, stderr=None, start_timeout=30.0, stop_timeout=8.0, idle_shutdown_timeout=_DEFAULT_IDLE_SHUTDOWN_TIMEOUT)
```

RAII handle around a child `tryll_server` process.

Use as a context manager (recommended), or call :meth:`stop` explicitly::

```
with ManagedServer.start(exe=Path("..."), port=9100) as srv:
    client = TryllClient.connect(srv.host, srv.port)
    ...
```

No executable discovery is performed — pass the path explicitly. The server is always launched with `--port <port>` so the caller's port takes precedence over whatever is in `server-config.json`, and (unless *idle_shutdown_timeout* is 0) with `--idle-shutdown-timeout <N>` so it self-exits once no session is using it — :meth:`stop` releases our handle but does not kill the process.

#### host

```
host
```

Host string to pass to :func:`TryllClient.connect`.

#### port

```
port
```

TCP port the server is listening on.

#### pid

```
pid
```

PID of the child process, or -1 if not running.

#### is_running

```
is_running
```

`True` while the child process is alive.

#### start

```
start(*, exe, host='127.0.0.1', port=9100, cwd=None, extra_args=(), stdout=None, stderr=None, start_timeout=30.0, stop_timeout=8.0, idle_shutdown_timeout=_DEFAULT_IDLE_SHUTDOWN_TIMEOUT)
```

Spawn the server and block until its TCP port is ready.

Parameters:

| Name                    | Type            | Description                                                                                                                                                                                                                                                                                                                                            | Default                                                           |
| ----------------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------- |
| `exe`                   | \`Path          | str\`                                                                                                                                                                                                                                                                                                                                                  | Path to tryll_server[.exe]. Required.                             |
| `host`                  | `str`           | Host string for the TCP ready-probe (default 127.0.0.1).                                                                                                                                                                                                                                                                                               | `'127.0.0.1'`                                                     |
| `port`                  | `int`           | TCP port; passed as --port to the server (default 9100).                                                                                                                                                                                                                                                                                               | `9100`                                                            |
| `cwd`                   | \`Path          | None\`                                                                                                                                                                                                                                                                                                                                                 | Working directory for the child process (defaults to exe.parent). |
| `extra_args`            | `Sequence[str]` | Additional CLI arguments appended after --port <port>.                                                                                                                                                                                                                                                                                                 | `()`                                                              |
| `stdout`                | \`Path          | None\`                                                                                                                                                                                                                                                                                                                                                 | File path for child stdout redirection (None = discard).          |
| `stderr`                | \`Path          | None\`                                                                                                                                                                                                                                                                                                                                                 | File path for child stderr redirection (None = discard).          |
| `start_timeout`         | `float`         | Seconds to wait for the TCP port to open (default 30).                                                                                                                                                                                                                                                                                                 | `30.0`                                                            |
| `stop_timeout`          | `float`         | Seconds to wait for clean exit on :meth:terminate (default 8).                                                                                                                                                                                                                                                                                         | `8.0`                                                             |
| `idle_shutdown_timeout` | `int`           | Seconds of idle (zero sessions, no in-flight downloads) after which the server self-exits; passed as --idle-shutdown-timeout <N> (default 60). :meth:stop does not kill the process — it relies on this timer — so pass 0 only when the caller has its own explicit reap strategy (e.g. the QA harness, which always calls :func:stop_server_process). | `_DEFAULT_IDLE_SHUTDOWN_TIMEOUT`                                  |

Returns:

| Name | Type              | Description                                              |
| ---- | ----------------- | -------------------------------------------------------- |
| `A`  | `'ManagedServer'` | class:ManagedServer whose port is accepting connections. |

Raises:

| Type                | Description                                     |
| ------------------- | ----------------------------------------------- |
| `FileNotFoundError` | If exe does not exist.                          |
| `TimeoutError`      | If the port does not open within start_timeout. |
| `OSError`           | If the process cannot be spawned.               |

#### stop

```
stop(*, timeout=None)
```

Release the process handle without killing the server.

The server self-exits via its idle-shutdown timer (managed mode). Callers that need an immediate, guaranteed stop (e.g. spawn succeeded but connect/handshake failed, so no session was ever created and nothing else can be relying on this process) should call :meth:`terminate` instead. The QA harness uses :func:`stop_server_process` directly for the same reason.

#### terminate

```
terminate(*, timeout=None)
```

Immediately hard-kill the child process, if still running.

Idempotent — safe to call more than once, including after :meth:`stop`. Intended for failure paths where the caller knows this process is theirs and nothing else can depend on it. Do not call this after a successful handshake, since another consumer could be relying on the server staying up until its own idle timeout.

### AgentWorkload

Bases: `IntEnum`

Whether a human ever waits on an agent's turns.

Purely a scheduling hint for the inference budget controller — it never changes node semantics. Set Background for world-logic/evaluator agents so their work is paced or deferred first under GPU pressure.

### AgentVariables

```
AgentVariables(client, agent_id, declared=None)
```

Mapping-like local mirror of one agent's declared per-agent variables.

Seed the mirror from the declaration passed to `create_agent`.

Parameters:

| Name       | Type                                   | Description                                                                                       | Default    |
| ---------- | -------------------------------------- | ------------------------------------------------------------------------------------------------- | ---------- |
| `client`   | `'TryllClient'`                        | Parent :class:~tryll_client.client.TryllClient.                                                   | *required* |
| `agent_id` | `int`                                  | Server-assigned agent id this store belongs to.                                                   | *required* |
| `declared` | `Optional[dict[str, tuple[int, Any]]]` | {name: (value_type, initial_value)} — the exact declaration sent in CreateAgentRequest.variables. | `None`     |

#### snapshot

```
snapshot()
```

Return a deep copy of every declared variable's current local value.

#### set

```
set(name, value)
```

Stage an `Assign` — *value*'s inferred type must match the declaration.

A plain Python `int` is accepted for a variable declared `float` (matches the filter grammar's int-to-float promotion); every other type mismatch raises immediately, before anything is queued for the wire.

#### add_to_set

```
add_to_set(name, element)
```

Stage an idempotent `AddToSet` — *name* must be a declared string set.

#### remove_from_set

```
remove_from_set(name, element)
```

Stage an idempotent `RemoveFromSet` — *name* must be a declared string set.

#### reset

```
reset(name)
```

Stage a `Reset` — restores the `create_agent`-time initial value.

#### stage_batch

```
stage_batch(ops)
```

Stage a batch atomically: validate+stage every op, or roll back all.

*ops* is a list of `(op_name, name, value)` where `op_name` is one of `assign` / `add_to_set` / `remove_from_set` / `reset` (the `value` is ignored for `reset`). On the first invalid op the mirror and staged queue are restored to their pre-call state and the error is re-raised, so a rejected batch leaves nothing dirty — mirroring the server's own validate-all-then-apply atomicity.

#### flush_if_dirty

```
flush_if_dirty(timeout=10.0)
```

Send every staged mutation as one batched `UpdateAgentVariablesRequest`.

Called automatically at the top of `send_message`, `send_message_async`, `resume`, `change_param`, and `change_params` — deliberately *not* before `cancel`. No-op when nothing is staged.

The batch is retained in-flight until acknowledged. On a transient `AgentBusy` it is re-staged behind any newer writes and the error is re-raised; on a hard rejection the affected names are re-synced from the server before re-raising, so a rejected batch never leaves the mirror silently diverged.

#### sync_from_server

```
sync_from_server(names=None, timeout=10.0)
```

Refresh the local mirror from the server's authoritative current values.

Names with a locally newer (staged or in-flight) write are skipped, so a server snapshot never clobbers an unsent local change.

Parameters:

| Name    | Type                  | Description                                                         | Default |
| ------- | --------------------- | ------------------------------------------------------------------- | ------- |
| `names` | `Optional[list[str]]` | Variable names to fetch; empty/None fetches all declared variables. | `None`  |

### VariableDecl

```
VariableDecl(value, allow_output_substitution=False)
```

One agent-variable value plus immutable declaration metadata.

______________________________________________________________________

## tryll_client.client

## tryll_client.client

TryllClient — synchronous TCP client for the Tryll server.

Threading model

- A background reader thread continuously receives frames and dispatches them to registered pending requests via threading.Event.
- The main (calling) thread sends requests and blocks on the Event.
- All access to \_pending is protected by \_lock.

### LoadModelResult

```
LoadModelResult(placement, backend)
```

What a :meth:`TryllClient.load_model` call actually got.

Both fields describe the load that happened, not the load that was asked for. `placement` is `Ram` when a `Vram` request could not be honoured (no GPU backend, or a CPU-only Sherpa ONNX Runtime build), and it is the *resident* value when another session already loaded the model the other way — placement is fixed at load time and deliberately is not part of the model cache key, so the first loader wins.

`backend` is the device actually serving the model (`"cuda"`, `"vulkan"`, `"rocm"`, `"cpu"`, or an ONNX Runtime provider), and is empty when the load resolved no backend. A client never chooses it: which GPU API is available is a host decision, process-wide and one-shot for llama.cpp and compile-time for Sherpa.

### TryllClient

```
TryllClient(sock)
```

Synchronous TCP session to the Tryll server.

Usage::

```
client = TryllClient.connect("127.0.0.1", 9100)
client.create_session([(ModelRole.Language, InferenceEngine.LlamaCpp)])
graph = GraphDescription().add_node(...).set_start_node(...)
# Exit fields (e.g. default_exit="") live on each node's typed params
agent = client.create_agent(graph)
response = agent.send_message("Hello")
agent.destroy()
client.shutdown()
```

#### connect

```
connect(host='127.0.0.1', port=9100, timeout=30.0)
```

Connect to the Tryll server and wait for ConnectionReady.

#### run_and_connect

```
run_and_connect(*, exe, host='127.0.0.1', port=9100, cwd=None, extra_args=(), stdout=None, stderr=None, start_timeout=30.0, stop_timeout=8.0, connect_timeout=30.0, idle_shutdown_timeout=60)
```

Spawn tryll_server and connect — recommended one-call factory.

Starts a local server process (passing `--port <port>` and, unless *idle_shutdown_timeout* is 0, `--idle-shutdown-timeout <N>` on the command line), waits for its TCP port to be ready, then opens a session. The returned :class:`ConnectedSession` owns both the server process and the client; use it as a context manager to guarantee clean teardown::

```
with TryllClient.run_and_connect(exe=Path("..."), port=9100) as session:
    session.client.create_session([(ModelRole.Language, InferenceEngine.LlamaCpp)])
    agent = session.client.create_agent(graph)
```

If the connected server's advertised `exe_path` does not match *exe* (a stranger raced us onto the port), the connection is torn down, the wrong process is hard-killed, and a fresh server is spawned on a new free port — up to a few attempts before raising.

Parameters:

| Name                    | Type            | Description                                                                                                                                                             | Default                                                           |
| ----------------------- | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| `exe`                   | \`Path          | str\`                                                                                                                                                                   | Path to tryll_server[.exe]. Required.                             |
| `host`                  | `str`           | Host for TCP probe and :meth:connect (default 127.0.0.1).                                                                                                               | `'127.0.0.1'`                                                     |
| `port`                  | `int`           | Port passed as --port to the server (default 9100).                                                                                                                     | `9100`                                                            |
| `cwd`                   | \`Path          | None\`                                                                                                                                                                  | Working directory for the child process (defaults to exe.parent). |
| `extra_args`            | `Sequence[str]` | Additional CLI arguments after --port <port>.                                                                                                                           | `()`                                                              |
| `stdout`                | \`Path          | None\`                                                                                                                                                                  | File path for child stdout (None = discard).                      |
| `stderr`                | \`Path          | None\`                                                                                                                                                                  | File path for child stderr (None = discard).                      |
| `start_timeout`         | `float`         | Seconds to wait for the TCP port to open (default 30).                                                                                                                  | `30.0`                                                            |
| `stop_timeout`          | `float`         | Seconds to wait for graceful server exit (default 8).                                                                                                                   | `8.0`                                                             |
| `connect_timeout`       | `float`         | Seconds to wait for :class:ConnectionReady (default 30).                                                                                                                | `30.0`                                                            |
| `idle_shutdown_timeout` | `int`           | Seconds of idle after which the launched server self-exits (default 60); see :meth:ManagedServer.start. Pass 0 only when the caller has its own explicit reap strategy. | `60`                                                              |

Returns:

| Name | Type                 | Description                                                  |
| ---- | -------------------- | ------------------------------------------------------------ |
| `A`  | `'ConnectedSession'` | class:ConnectedSession whose server is running and client is |
|      | `'ConnectedSession'` | connected.                                                   |

Raises:

| Type                | Description                                                                                 |
| ------------------- | ------------------------------------------------------------------------------------------- |
| `FileNotFoundError` | If exe does not exist.                                                                      |
| `TimeoutError`      | If the port does not open within start_timeout.                                             |
| `TryllError`        | If the session handshake fails, or identity verification keeps failing after a few retries. |

#### set_session_locale

```
set_session_locale(locale, timeout=30.0)
```

Change the session language without tearing the session down.

Same BCP-47 grammar as :meth:`create_session`'s `locale`; an empty string *clears* the locale, restoring pre-locale behaviour everywhere.

```
Applies to every agent, existing and future, from its next turn: TTS
language and the locale template keys are read per turn, not captured.
A new voice input picks up the new STT language hint; one already open
keeps its recognizer. Storage paths re-resolve on the next load, but
content already bound to a node (a Retrieve KB, canned lines) does NOT
swap - those params are structural. Use this for the voice + template
half of a language change; switching authored content still means
recreating agents.
```

Not turn-atomic: a turn already running may render its prompt under the old locale and synthesize under the new one. Send it between turns if that matters.

Parameters:

| Name      | Type    | Description                                 | Default    |
| --------- | ------- | ------------------------------------------- | ---------- |
| `locale`  | `str`   | BCP-47 tag ("de", "pt-BR"), or '' to clear. | *required* |
| `timeout` | `float` | Maximum seconds to wait for the server Ack. | `30.0`     |

Raises:

| Type         | Description                                     |
| ------------ | ----------------------------------------------- |
| `TryllError` | InvalidLocale (2004) when the tag is malformed. |

#### create_session

```
create_session(engines=None, game_name='', timeout=30.0, storage_data_folder='', locale='', placement_hints=None, hotword_hints=None)
```

Create the logical session for this connection (mandatory, one-shot).

Sends CreateSessionRequest and waits for CreateSessionResponse, which carries the server-allocated `session_id` (stored on :attr:`session_id`). Must be called exactly once after :meth:`connect` and before :meth:`create_agent` or any model/storage/voice call — those are rejected with `SessionNotReady` until the session exists. A second call is rejected by the server with `SessionAlreadyExists`.

Parameters:

| Name                  | Type                                       | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | Default                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| --------------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `engines`             | \`'Optional\[Sequence\[tuple\[ModelRole    | int, InferenceEngine                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | int\]\]\]'\`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `game_name`           | `str`                                      | Integration identifier for telemetry grouping (e.g. "qa-and-eval"). Leave empty for anonymous sessions.                                                                                                                                                                                                                                                                                                                                                                                                             | `''`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `timeout`             | `float`                                    | Maximum seconds to wait for the server response.                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | `30.0`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `placement_hints`     | \`'Optional\[Mapping\[str, MemoryPlacement | int\]\]'\`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | Optional per-model memory placement, e.g. {"Llama 3.1 8B": MemoryPlacement.Ram}. Sparse: a model you do not list means :attr:MemoryPlacement.Auto. These are hints — a host that cannot honour one downgrades it and reports what it actually did, so read the result from :meth:load_model rather than assuming. LoadModelRequest.placement overrides a hint for a single load.                                                                                                                                                                                                                                                                                                                                                   |
| `hotword_hints`       | \`'Optional\[Mapping\[str, HotwordBias     | int\]\]'\`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | Optional per-model hotword biasing, e.g. {"Parakeet TDT 0.6B v3 (int8)": HotwordBias.On}. Sparse: a model you do not list follows its catalog entry, which biases when that entry sets a non-zero stt_hotwords_score. Unlike placement_hints this is not downgradable. Biasing needs beam-search decoding, which a recognizer fixes when it is built, so this selects which recognizer the session gets — and two sessions that disagree about one model hold two of them. On costs a few percent of decode time and changes that model's transcripts whether or not you pass a lexicon; Off makes :meth:create_voice_input reject a hotwords_storage_path with HotwordBiasDisabled (4106) rather than accept it and bias nothing. |
| `storage_data_folder` | `str`                                      | Optional per-session storage data folder. Relative storage paths in node params (and voice-input hotwords) resolve against this folder. Empty (default) means the server falls back to its configured storage_root.                                                                                                                                                                                                                                                                                                 | `''`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `locale`              | `str`                                      | Optional session language as a BCP-47 tag ("de", "de-DE", "pt-BR"). One setting the whole voice + text stack derives from: the default tts_lang for every GenerateAndSpeak/Speak node, the language hint for multilingual STT models, loc/<locale>/ siblings prefix relative storage paths prefer, and the language Mustache key a prompt can write a "reply in X" directive against. Empty (default) leaves every one of those at its pre-locale behaviour. A malformed tag is rejected with InvalidLocale (2004). | `''`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |

#### set_inference_throttle

```
set_inference_throttle(level)
```

Report how hard the server should yield the GPU back to the game.

Fire-and-forget: no response, no error surfaced locally. Never sent automatically — a session that never calls this runs unthrottled, identically to a client on an older protocol version.

Parameters:

| Name    | Type    | Description                                                                                                 | Default    |
| ------- | ------- | ----------------------------------------------------------------------------------------------------------- | ---------- |
| `level` | `float` | 0 = full speed (default), 1 = maximum yielding. Clamped server-side; out-of-range values do not raise here. | *required* |

#### create_string_storage

```
create_string_storage(name, strings=None, file_path=None, timeout=10.0)
```

Create a named StringStorage on the server.

Provide either *strings* (inline list) or *file_path* (server-side file path). The storage can then be referenced by name in node params via `string_storage`.

#### create_keyed_string_storage

```
create_keyed_string_storage(name, keys, values, kind=StringStorageKind.Map, timeout=10.0)
```

Create a Map or Multimap-kind StringStorage on the server.

*keys* and *values* must be the same length. For `Map` kind all keys must be unique.

The storage can then be referenced by name in `IntentToInstructionNode` (and other keyed-consumer nodes) via the `string_storage` param.

#### destroy_string_storage

```
destroy_string_storage(name, timeout=10.0)
```

Destroy a named StringStorage on the server.

Nodes that already hold the storage are unaffected.

#### create_embedded_string_storage

```
create_embedded_string_storage(name, config_path=None, strings=None, embedding_model=None, timeout=None)
```

Create a named EmbeddedStringStorage on the server.

Path A: supply *config_path* (server-side `*.json`). Path B: supply *strings* (inline list) + *embedding_model*. Returns an EmbeddedStorageInfo with record_count and embedding_dim.

#### destroy_embedded_string_storage

```
destroy_embedded_string_storage(name, timeout=10.0)
```

Destroy a named EmbeddedStringStorage on the server.

Nodes that already hold the storage are unaffected.

#### create_voice_input

```
create_voice_input(model_name, sample_rate=16000, channels=1, bits_per_sample=16, vad_threshold=0.5, vad_min_silence_ms=1000, vad_speech_pad_ms=250, hotwords_storage_path='', hotwords_score=1.5, timeout=30.0)
```

Create a server-side VoiceInput session for speech-to-text.

Returns a :class:`~tryll_client.voice_input.VoiceInput` handle managing the server-side session. Use as a context manager (recommended) for guaranteed cleanup.

Parameters:

| Name                    | Type    | Description                                                                                                                                                                                                                                                            | Default    |
| ----------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
| `model_name`            | `str`   | Catalog name of the STT model, e.g. "Parakeet TDT 0.6B v3 (int8)".                                                                                                                                                                                                     | *required* |
| `sample_rate`           | `int`   | Sample rate of audio buffers you will push (Hz).                                                                                                                                                                                                                       | `16000`    |
| `channels`              | `int`   | Channel count (1 = mono, typical for mic capture).                                                                                                                                                                                                                     | `1`        |
| `bits_per_sample`       | `int`   | Bit depth of raw PCM samples (only 16-bit is supported).                                                                                                                                                                                                               | `16`       |
| `vad_threshold`         | `float` | Silero VAD speech-probability threshold (0.0–1.0).                                                                                                                                                                                                                     | `0.5`      |
| `vad_min_silence_ms`    | `int`   | Silence duration (ms) that closes a speech segment.                                                                                                                                                                                                                    | `1000`     |
| `vad_speech_pad_ms`     | `int`   | Padding (ms) added around detected speech.                                                                                                                                                                                                                             | `250`      |
| `hotwords_storage_path` | `str`   | Relative path (resolved against the session storage folder) to a StringStorage (kind=List) config whose phrases the STT decoder will bias toward. Loaded on demand — no prior create_string_storage needed. Pass an empty string (default) to disable hotword biasing. | `''`       |
| `hotwords_score`        | `float` | How strongly matched phrases are weighted against one another during beam search. Measured on Parakeet TDT 0.6B v3: 1.8 recovered 5 of 8 target terms, 2.5 recovered 6 of 8. A lexicon line may override this for itself by ending with :<number>.                     | `1.5`      |
| `timeout`               | `float` | Maximum seconds to wait for the server response.                                                                                                                                                                                                                       | `30.0`     |

Raises:

| Type         | Description                                                                                   |
| ------------ | --------------------------------------------------------------------------------------------- |
| `TryllError` | If the model cannot be found, the hotwords storage file is missing, or the request times out. |

#### create_agent

```
create_agent(graph, enable_diagnostics=False, timeout=None, maintain_dialogue_history=True, variables=None, kv_cache_initialization=AgentKvCacheInitialization.AllocateOnly, workload=AgentWorkload.Interactive)
```

Create a server-side agent and return a proxy handle.

Parameters:

| Name                        | Type               | Description                                                                                                                                                                                                                      | Default                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| --------------------------- | ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `graph`                     | `GraphDescription` | Fully-built graph description.                                                                                                                                                                                                   | *required*                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `enable_diagnostics`        | `bool`             | When True the server serialises per-node execution data into TurnComplete.debug_info for every turn.                                                                                                                             | `False`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `timeout`                   | \`float            | None\`                                                                                                                                                                                                                           | Maximum seconds to wait for the response. Defaults to 30 s.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `maintain_dialogue_history` | `bool`             | When True (default) the agent keeps its full dialogue history. When False the history is discarded after each turn — a stateless agent, useful for technical/classification agents that should not store or project prior turns. | `True`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `variables`                 | \`dict[str, Any]   | None\`                                                                                                                                                                                                                           | Declares this agent's per-agent variables — a {name: initial_value} mapping. Values may be consumed from Mustache templates as {{var.<name>}} and the Retrieve/ClassifyIntent filter grammar as {"var": "<name>"}. Values may instead be VariableDecl(initial_value, allow_output_substitution=True) to permit output marker substitution for that declaration. Plain values default the permission to False. The wire type is inferred from each value (bool/int/float/str/ set or list of str) and locked for the agent's lifetime; this is the complete declaration — later writes to undeclared names raise TryllError (3013). |
| `workload`                  | \`AgentWorkload    | int\`                                                                                                                                                                                                                            | :class:AgentWorkload (or its ordinal). Whether a human ever waits on this agent's turns — pass AgentWorkload.Background for world-logic/evaluator agents so the inference scheduler paces or defers their work first under GPU pressure. Purely a scheduling hint; it never changes node semantics. Fixed for the agent's lifetime — it is not a node parameter, so change_param cannot alter it; recreate the agent instead.                                                                                                                                                                                                      |

#### list_models

```
list_models(timeout=10.0)
```

Request every model in the server's catalog, each reporting its engine.

#### get_memory_consumption

```
get_memory_consumption(timeout=10.0)
```

Pull a server-wide RAM/VRAM snapshot.

Session-scoped (must follow create_session) but the numbers cover every session on the process — the same view as the server monitor. Never fails with AgentBusy.

#### load_model

```
load_model(model_name, timeout=300.0, placement=None)
```

Explicitly load and pin a model into memory.

The model stays in memory until :meth:`unload_model` is called, regardless of whether any agents are using it. Raises :class:`TryllError` if the model cannot be resolved or loaded.

Parameters:

| Name        | Type               | Description | Default |
| ----------- | ------------------ | ----------- | ------- |
| `placement` | \`'MemoryPlacement | int         | None'\` |

Returns:

| Name       | Type              | Description                                                           |
| ---------- | ----------------- | --------------------------------------------------------------------- |
|            | `LoadModelResult` | class:LoadModelResult — where the model actually ended up and which   |
|            | `LoadModelResult` | backend is serving it. Read this rather than assuming the request was |
| `honoured` | `LoadModelResult` | a Vram request on a host with no GPU backend reports                  |
|            | `LoadModelResult` | Ram, and a model another session already loaded reports its           |
|            | `LoadModelResult` | resident placement. A mismatch against an explicit request is also    |
|            | `LoadModelResult` | logged at warning level.                                              |

#### unload_model

```
unload_model(model_name, timeout=30.0)
```

Unpin a previously pinned model.

If no agents are currently using the model it is freed immediately; otherwise it will be freed when the last agent using it is destroyed.

#### download_model

```
download_model(model_name, on_progress=None, timeout=1800.0)
```

Initiate a model download on the server and block until complete.

*on_progress* is called with (model_name, bytes_downloaded, total_bytes, percent) for each progress frame received. Raises TryllError on failure.

#### send_debug_command

```
send_debug_command(command, payload='', *, await_response=True, timeout=10.0)
```

Send a dev-only debug command to the server.

The server handles this only in non-Production builds with `debug_commands.enabled = true`; otherwise it returns a TryllError.

`command` is a verb (e.g. `"crash"`); `payload` is verb-specific (for `crash` it is the mode token: `nullderef` | `abort` | `throw` | `throw_posted` | `stackoverflow` | `fastfail`; empty defaults to `nullderef`).

Set `await_response=False` for fatal verbs like `crash` that kill the server and never reply — the call sends the frame and returns immediately; the caller detects success via the ensuing TCP disconnect. When `await_response` is True, returns the server's `result` string.

#### wait_for_disconnect

```
wait_for_disconnect(timeout=10.0)
```

Block until the server drops the connection, or *timeout* elapses.

The reader thread exits when the socket reaches EOF / resets (e.g. the server process died), so joining it is a reliable disconnect signal. Returns True if the connection dropped within *timeout*. Used after a fatal debug command (`crash`) whose only success signal is the drop.

#### shutdown

```
shutdown()
```

Close the socket and stop the reader thread.

### ConnectedSession

```
ConnectedSession(*, server, client)
```

RAII wrapper that owns a :class:`~tryll_client.managed_server.ManagedServer` and a connected :class:`TryllClient`.

Obtain via :meth:`TryllClient.run_and_connect`. Use as a context manager (recommended) to guarantee teardown in the right order — client first, then server::

```
with TryllClient.run_and_connect(exe=Path("..."), port=9100) as session:
    session.client.create_session([(ModelRole.Language, InferenceEngine.LlamaCpp)])
    agent = session.client.create_agent(graph)
    print(agent.send_message("hi"))
```

#### client

```
client
```

The connected :class:`TryllClient`.

#### server

```
server
```

The running :class:`~tryll_client.managed_server.ManagedServer`.

#### shutdown

```
shutdown()
```

Shut the client down, then stop the server. Idempotent.

______________________________________________________________________

## tryll_client.agent

## tryll_client.agent

AgentProxy — client-side handle for a server-side agent.

An :class:`AgentProxy` is returned by :meth:`tryll_client.TryllClient.create_agent` and represents a single server-side agent running a pre-built workflow graph. The proxy is single-session and not thread-safe beyond what the parent :class:`TryllClient` provides.

### TurnResult

```
TurnResult(text, status, ttft_s, tokens_generated, debug_info)
```

Outcome of a turn started with :meth:`AgentProxy.send_message_async`.

Delivered by resolving the returned :class:`concurrent.futures.Future` once `TurnComplete` arrives. `status` distinguishes a normal completion from a cancelled one (:class:`~tryll_client.graph.TurnStatus`).

### ToolResult

```
ToolResult(call_id, result)
```

One model-visible result correlated to a tool-call `call_id`.

### IntentClassifiedEvent

```
IntentClassifiedEvent(node_name, intent, record_id, record_index, distance, not_found_reason)
```

Embedding ClassifyIntent payload (`event_type="intent_classified"`).

### IntentLlmLabelProb

```
IntentLlmLabelProb(label, prob)
```

One `<label>.prob` pair from ClassifyIntentLLM.

### IntentLlmClassifiedEvent

```
IntentLlmClassifiedEvent(node_name, intent, top_prob, second_prob, not_found_reason, label_probs)
```

ClassifyIntentLLM payload (`event_type="intent_llm_classified"`).

### AgentProxy

```
AgentProxy(client, agent_id, node_baselines=None, variables=None)
```

Handle for one server-side agent created via `TryllClient.create_agent()`.

The proxy exposes the user-facing turn API — :meth:`send_message` and :meth:`destroy` — and caches diagnostics from the last completed turn on `last_*` properties.

Bind the proxy to its owning client and server-side agent id.

Parameters:

| Name             | Type                           | Description                                                                               | Default                                                                                                                                                            |
| ---------------- | ------------------------------ | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `client`         | `'TryllClient'`                | Parent :class:TryllClient used to send and receive wire messages on behalf of this agent. | *required*                                                                                                                                                         |
| `agent_id`       | `int`                          | Server-assigned agent identifier returned in the CreateAgentResponse.                     | *required*                                                                                                                                                         |
| `node_baselines` | \`dict[str, 'NodeParamsBase']  | None\`                                                                                    | Deep-copied params per graph node name, captured at create_agent time. Used by :meth:change_param to apply single-field mutations without caller-side bookkeeping. |
| `variables`      | \`dict\[str, tuple[int, Any]\] | None\`                                                                                    | {name: (value_type, initial_value)} declared at create_agent time, used to seed :attr:variables.                                                                   |

#### agent_id

```
agent_id
```

Server-assigned agent identifier for this proxy.

#### last_debug_info

```
last_debug_info
```

JSON diagnostics string from the most recent `send_message` call.

Returns:

| Type  | Description                                                   |
| ----- | ------------------------------------------------------------- |
| `str` | Server-produced JSON document attached to TurnComplete        |
| `str` | when diagnostics were enabled on agent creation; otherwise an |
| `str` | empty string. Empty string also before the first turn.        |

#### last_ttft_s

```
last_ttft_s
```

Time-to-first-token in seconds for the most recent turn.

Returns:

| Type    | Description |
| ------- | ----------- |
| \`float | None\`      |
| \`float | None\`      |
| \`float | None\`      |

#### last_answer_chunk_count

```
last_answer_chunk_count
```

Number of `AnswerText` chunks received for the last turn.

Typically one chunk per generated token when streaming; useful for verifying the stream was delivered incrementally.

#### last_tokens_generated

```
last_tokens_generated
```

Server-reported generated token count for the last turn.

Authoritative for both streaming and non-streaming modes. Zero if the server has not yet completed a turn.

#### paused_node

```
paused_node
```

Node this agent's turn is currently paused after; `''` when not paused.

A non-empty value means the server is holding the turn open until :meth:`resume` (or :meth:`cancel`) — until then :meth:`send_message` keeps failing with `AgentBusy` (3004).

#### set_on_answer_text

```
set_on_answer_text(cb)
```

Register (or clear) a persistent callback for streaming text chunks.

The callback is invoked on the reader thread for every `AnswerText` frame received for this agent — including frames from server-initiated turns (e.g. voice autosend after `VoiceInput.end_utterance`).

The callback signature is `(node_name: str, text: str, is_delta: bool, is_final: bool)`. `node_name` identifies which node in the workflow graph produced this text (multi-sender attribution); existing callers that only need the text can ignore it. Must return quickly and must not call blocking client methods.

Call with `None` to unregister the current callback.

Parameters:

| Name | Type                                       | Description | Default                                                                        |
| ---- | ------------------------------------------ | ----------- | ------------------------------------------------------------------------------ |
| `cb` | \`Callable\[[str, str, bool, bool], None\] | None\`      | Callable invoked with (node_name, text, is_delta, is_final), or None to clear. |

#### set_on_turn_complete

```
set_on_turn_complete(cb)
```

Register (or clear) a persistent callback for turn-complete notifications.

The callback is invoked on the reader thread once per `TurnComplete` frame for this agent. Covers both client-initiated turns (via :meth:`send_message`) and server-initiated turns (voice autosend).

The callback signature is `(status: int, debug_info: str, tokens_generated: int)` where `status` maps to the wire `TurnStatus` enum (0 = Ok, 1 = Error, 2 = Cancelled).

Call with `None` to unregister the current callback.

Parameters:

| Name | Type       | Description | Default                                                                         |
| ---- | ---------- | ----------- | ------------------------------------------------------------------------------- |
| `cb` | \`Callable | None\`      | Callable invoked with (status, debug_info, tokens_generated), or None to clear. |

#### set_on_tool_call

```
set_on_tool_call(cb)
```

Register (or clear) a callback for tool-call notifications.

The callback is invoked on the reader thread for every `NodeEvent` frame with `event_type == "tool_call"` the server sends for this agent — i.e. when the graph has a `ToolCall` node with a notifying disposition (Notify / NotifyAndAcknowledge / Pause / PauseAndAcknowledge / AwaitResult). It must return quickly and must not call any blocking :class:`TryllClient` or :class:`AgentProxy` methods.

The callback signature is `(tool_name: str, arguments_json: str)` where `arguments_json` is a compact JSON object, e.g. `'{"city": "Berlin"}'`. Parse it with :func:`json.loads` as needed.

Call with `None` to unregister the current callback.

Parameters:

| Name | Type                           | Description | Default                                                              |
| ---- | ------------------------------ | ----------- | -------------------------------------------------------------------- |
| `cb` | \`Callable\[[str, str], None\] | None\`      | Callable invoked with (tool_name, arguments_json), or None to clear. |

#### set_on_tool_call_with_id

```
set_on_tool_call_with_id(cb)
```

Register a call-ID-aware tool-call callback.

The callback receives `(call_id, tool_name, arguments_json)` on the reader thread. Echo `call_id` in a :class:`ToolResult` passed through the keyword-only `tool_results` argument of :meth:`resume_async` (preferred from this callback) or :meth:`resume` (only from a non-reader thread). The legacy :meth:`set_on_tool_call` callback remains supported.

#### set_on_error

```
set_on_error(cb)
```

Register (or clear) a per-agent client-originated error callback.

Fired on the reader thread when a user callback (tool_call, paused, etc.) raises. Does not replace server `ErrorResponse` delivery for pending requests. The error callback itself must not raise; secondary failures are swallowed to protect the reader.

#### set_on_intent_classified

```
set_on_intent_classified(cb)
```

Register (or clear) the embedding ClassifyIntent callback.

Fired on the reader thread for every `NodeEvent` with `event_type == "intent_classified"`. Call with `None` to unregister.

#### set_on_intent_llm_classified

```
set_on_intent_llm_classified(cb)
```

Register (or clear) the ClassifyIntentLLM callback.

Fired on the reader thread for every `NodeEvent` with `event_type == "intent_llm_classified"`. Call with `None` to unregister.

#### set_on_paused

```
set_on_paused(cb)
```

Register (or clear) a callback for pause notifications.

Fired on the reader thread for every `NodeEvent` with `event_type == "paused"` — emitted by the executor when it pauses the turn between nodes (see the `Pause` node and pausing `ToolCallDisposition` values). While paused, the agent still counts as busy for :meth:`send_message`, but :meth:`change_params` / :meth:`change_param` are allowed.

The callback signature is `(node_name: str, pending_exit: str)` where `pending_exit` is the exit route that will be taken on a plain (non-jump) :meth:`resume`.

Call with `None` to unregister.

#### set_on_tts_audio_format

```
set_on_tts_audio_format(cb)
```

Register (or clear) a callback for the TTS audio format frame.

Emitted by the server exactly once per turn that produces TTS audio, before the first :meth:`set_on_tts_audio` callback. The callback signature is `(node_name: str, sample_rate: int, channels: int, bits_per_sample: int)` — `node_name` is the producing node's name (multi-sender attribution; empty string if the server omitted it). Fired on the reader thread — must return quickly and must not call blocking client methods.

Call with `None` to unregister.

#### set_on_tts_audio

```
set_on_tts_audio(cb)
```

Register (or clear) a callback for streaming TTS audio chunks.

Fired on the reader thread for every `TtsAudioFrame` received for this agent. The callback signature is `(node_name: str, pcm: memoryview)` — `node_name` is the producing node's name (multi-sender attribution; matches the preceding :meth:`set_on_tts_audio_format` call for the same producer). The `pcm` argument is a zero-copy `memoryview` over int16 LE PCM at the format declared by the preceding format callback. The view is valid only during the call — copy with `bytes(pcm)` or `np.frombuffer(pcm, dtype=np.int16).copy()` if needed.

`TtsAudioFrame` carries no `is_final` flag — end-of-stream is signalled by `TurnComplete`. Wire :meth:`set_on_turn_complete` (or close your sink in the surrounding `with`-block) to flush.

Call with `None` to unregister.

#### set_on_node_event

```
set_on_node_event(cb)
```

Register (or clear) a generic `NodeEvent` fallback callback.

Fired on the reader thread for any `NodeEvent` whose `event_type` is unrecognised or whose typed callback (e.g. :meth:`set_on_tool_call`, :meth:`set_on_intent_classified`) is not registered. When a typed callback handles the event, this one is NOT invoked for that event.

The callback signature is `(node_name: str, event_type: str, kv_pairs: list[tuple[str, str]])`.

Call with `None` to unregister.

#### send_message

```
send_message(text, timeout=120.0)
```

Send a user message and return the complete assistant response.

Blocks until `TurnComplete` is received from the server, accumulating all `AnswerText` chunks into a single string. Diagnostics from `TurnComplete` (`debug_info`, `tokens_generated`) are cached on the `last_*` properties.

Parameters:

| Name      | Type    | Description                               | Default    |
| --------- | ------- | ----------------------------------------- | ---------- |
| `text`    | `str`   | User-turn text to send to the agent.      | *required* |
| `timeout` | `float` | Maximum seconds to wait for TurnComplete. | `120.0`    |

Returns:

| Type  | Description                                    |
| ----- | ---------------------------------------------- |
| `str` | The full concatenated assistant response text. |

Raises:

| Type         | Description                                                                                    |
| ------------ | ---------------------------------------------------------------------------------------------- |
| `TryllError` | On server-reported errors, decode failures, or if timeout elapses before TurnComplete arrives. |

#### send_message_async

```
send_message_async(text)
```

Send a user message without blocking; return a Future for the result.

Unlike :meth:`send_message`, this returns immediately so the caller can act mid-turn (e.g. watch streaming callbacks and call :meth:`cancel`). The Future resolves with a :class:`TurnResult` on `TurnComplete` (including a cancelled turn, where `status == TurnStatus.Cancelled`) or raises :class:`~tryll_client.errors.TryllError` on a server error.

Await it via `fut.result(timeout)` (raises `concurrent.futures.TimeoutError` if the turn does not finish in time). Streaming chunks still arrive via the `set_on_answer_text` / `set_on_tts_audio` callbacks.

Note: use :meth:`cancel` to cancel the turn — do **not** call `fut.cancel()`, which only cancels a scheduled call, not the server-side turn.

#### change_param

```
change_param(node_name, path, value, timeout=30.0)
```

Apply a single-field mutation using a dotted attribute path.

Clones the baseline params captured at `create_agent`, coerces *value* to the leaf field type (when *value* is a string), assigns via :func:`~tryll_client._generated.node_params_codec.set_dotted`, and sends the full params object. On success the baseline is updated so later mutations compose.

Example::

```
agent.change_param("generate", "sampling.temperature", "0.5")
```

Parameters:

| Name        | Type    | Description                                                            | Default    |
| ----------- | ------- | ---------------------------------------------------------------------- | ---------- |
| `node_name` | `str`   | Instance name of the target node.                                      | *required* |
| `path`      | `str`   | Dotted path to the field (e.g. "sampling.temperature").                | *required* |
| `value`     | `Any`   | New value (string from JSON/dialog scripts, or a native Python value). | *required* |
| `timeout`   | `float` | Maximum seconds to wait for the server Ack.                            | `30.0`     |

Raises:

| Type         | Description                                                              |
| ------------ | ------------------------------------------------------------------------ |
| `TryllError` | If node_name is unknown (code 3005) or the server reports another error. |

#### change_params

```
change_params(node_name, params, timeout=30.0)
```

Apply a typed node-parameter update to a workflow node at runtime.

The typed params object must match the concrete type of the target node. Structural fields must match the create-time values or the server returns `ParamNotMutable`. On success, updates the internal baseline used by :meth:`change_param`.

Parameters:

| Name        | Type               | Description                                 | Default    |
| ----------- | ------------------ | ------------------------------------------- | ---------- |
| `node_name` | `str`              | Instance name of the target node.           | *required* |
| `params`    | `'NodeParamsBase'` | Typed node parameters (generated DTO).      | *required* |
| `timeout`   | `float`            | Maximum seconds to wait for the server Ack. | `30.0`     |

Raises:

| Type         | Description                                                                                                            |
| ------------ | ---------------------------------------------------------------------------------------------------------------------- |
| `TryllError` | On server-reported errors (AgentBusy 3004, UnknownNode 3005, ParamNotMutable 3006, InvalidParamValue 3007) or timeout. |

#### destroy

```
destroy(timeout=30.0)
```

Request agent destruction on the server and wait for `Ack`.

After this call returns, the proxy is no longer usable; further `send_message` calls will raise :class:`TryllError`.

Parameters:

| Name      | Type    | Description                                   | Default |
| --------- | ------- | --------------------------------------------- | ------- |
| `timeout` | `float` | Maximum seconds to wait for the Ack response. | `30.0`  |

Raises:

| Type         | Description                           |
| ------------ | ------------------------------------- |
| `TryllError` | On server-reported errors or timeout. |

#### cancel

```
cancel(mode=0, timeout=30.0)
```

Cooperatively cancel the agent's in-flight turn and wait for `Ack`.

The active `send_message` ends with `TurnStatus.Cancelled`. Call this from a different thread than the blocking `send_message` to interrupt a turn mid-flight. No-op server-side if the agent is idle.

Parameters:

| Name      | Type    | Description                                                                                                                                                  | Default |
| --------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------- |
| `mode`    | `int`   | :class:~tryll_client.graph.CancelMode — StopAndKeep (0, default) keeps the partial interaction; StopAndDiscard (1) removes it as if the turn never happened. | `0`     |
| `timeout` | `float` | Maximum seconds to wait for the Ack response.                                                                                                                | `30.0`  |

Raises:

| Type         | Description                           |
| ------------ | ------------------------------------- |
| `TryllError` | On server-reported errors or timeout. |

#### resume

```
resume(resume_node='', timeout=30.0, *, tool_results=())
```

Resume a turn that is currently paused and wait for `Ack`.

**Not safe on the reader thread** (tool-call / paused callbacks): blocking here deadlocks the sole frame reader. Use :meth:`resume_async` from those callbacks instead.

Parameters:

| Name           | Type                   | Description                                                                                                                       | Default                                                                                                                                                                                                                                              |
| -------------- | ---------------------- | --------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `resume_node`  | `str`                  | Empty continues via the paused node's pending exit route; non-empty jumps to that node by name (must exist in the agent's graph). | `''`                                                                                                                                                                                                                                                 |
| `timeout`      | `float`                | Maximum seconds to wait for the Ack response.                                                                                     | `30.0`                                                                                                                                                                                                                                               |
| `tool_results` | \`Sequence\[ToolResult | tuple[str, str]\]\`                                                                                                               | Complete result batch for the paused tool-call pass. Each result must echo the call_id surfaced by :meth:set_on_tool_call_with_id. Typed :class:ToolResult values are preferred; (call_id, result) pairs are accepted for codec-level compatibility. |

Raises:

| Type         | Description                                                                                                                                     |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `TryllError` | AgentNotPaused (3012) if the agent is not currently paused, UnknownNode (3005) if resume_node does not name a node in the graph, or on timeout. |

#### resume_async

```
resume_async(resume_node='', *, tool_results=())
```

Resume a paused turn without blocking; returns a Future for the Ack.

Safe to call from reader-thread callbacks. The Future resolves to `None` on success or raises :class:`TryllError` on failure.

#### prefill_kv_cache

```
prefill_kv_cache(timeout=30.0)
```

Synchronise eligible language-model contexts to the reusable prefix.

#### evict_kv_cache

```
evict_kv_cache(timeout=30.0)
```

Release eligible raw contexts while retaining agent state.

#### get_kv_cache_status

```
get_kv_cache_status(timeout=30.0)
```

Return aggregate residency and reusable-prefix state.

#### append_interactions

```
append_interactions(interactions, timeout=30.0)
```

Append scripted user/assistant interactions without running the graph.

Each item is `(user_message, assistant_message)`. Empty / `None` strings omit that side; both empty → skipped. Idle-only (`AgentBusy` 3004 while running/paused).

Returns:

| Type  | Description                               |
| ----- | ----------------------------------------- |
| `int` | Number of interactions actually appended. |

#### remove_interactions_from_end

```
remove_interactions_from_end(count, timeout=30.0)
```

Remove whole interactions from the end of the dialog.

`count == 0` is a no-op; `count >= size` clears all. Idle-only.

Returns:

| Type  | Description                              |
| ----- | ---------------------------------------- |
| `int` | Number of interactions actually removed. |

Raises:

| Type         | Description                                            |
| ------------ | ------------------------------------------------------ |
| `ValueError` | If count is negative.                                  |
| `TryllError` | On server-reported errors (AgentBusy 3004) or timeout. |

#### export_dialog

```
export_dialog(timeout=30.0)
```

Return the agent's completed dialog as an owned snapshot. Idle-only.

#### import_dialog

```
import_dialog(snapshot, compatibility=0, timeout=30.0)
```

Replace the agent's dialog with *snapshot*. Idle-only. Never appends.

______________________________________________________________________

## tryll_client.graph

## tryll_client.graph

GraphDescription builder and associated enums / dataclasses.

Mirrors the C++ `Tryll::Client::GraphDescription` / `InferenceEngine` / `NodeType` types. Enum values must stay in sync with `server/schema/messages.fbs`.

This module exposes:

- :class:`GraphDescription` — fluent builder for a workflow graph sent to the server on `CreateAgent`.
- The enums used by graph descriptions: :class:`InferenceEngine`, :class:`ModelStatus`.
- The typed node-parameter DTOs from `tryll_client._generated.node_params`.

### NodeParamsBase

Marker base for generated node-parameter DTOs.

### SendAnswer

Bases: `IntEnum`

Controls answer-text delivery to the client. Replaces the old `stream: bool`.

### HistoryRole

Bases: `IntEnum`

Controls whether/how a producer's slot output is replayed in later turns.

### KvCacheType

Bases: `IntEnum`

llama.cpp KV cache dtype. Inherit = model variant, then q8_0.

### RetrievalMode

Bases: `IntEnum`

RetrieveNode search path: dense HNSW, BM25 lexical, or RRF hybrid.

### BranchTest

Bases: `IntEnum`

Predicate policy for BranchNode.

### NotifyClient

Bases: `IntEnum`

When a ClassifyIntent / ClassifyIntentLLM node emits a NodeEvent.

### NodeType

Bases: `IntEnum`

Node type ordinal. Mirrors the NodeParams union order in nodes.fbs (0-based; the wire discriminator is this value + 1).

The union discriminant in `NodeParams` supersedes this for the wire format; `NodeType` is kept for client-side ergonomics and generated convenience APIs.

### InferenceEngine

Bases: `IntEnum`

Inference backend. Mirrors the FlatBuffers `InferenceEngine` enum.

### ModelRole

Bases: `IntEnum`

What a model is for. Mirrors the FlatBuffers `ModelRole` enum.

Used to qualify an engine declaration in :meth:`TryllClient.create_session`. The role is redundant for lookup — the server reads each model's engine from its own catalog entry — and exists so a rejection can name what the mismatch actually is.

### MemoryPlacement

Bases: `IntEnum`

Where a model's weights should live. Mirrors `MemoryPlacement`.

Always a *hint*: a host that cannot honour it downgrades and reports what it did, so read the resolved value back rather than assuming. Which GPU API provides VRAM (CUDA / Vulkan / HIP) is a host decision and is deliberately not expressible here.

`Auto` is all-or-nothing in this version — device memory when the host has a GPU, system memory otherwise. It does not size offload to free memory.

### HotwordBias

Bases: `IntEnum`

Whether a session wants hotword biasing for one STT model.

Overrides the model catalog's `stt_hotwords_score`. Unlike :class:`MemoryPlacement` this is not a downgradable hint: biasing needs beam-search decoding, which a recognizer fixes when it is constructed, so the answer selects *which recognizer* the session is served. Two sessions that disagree about one model hold two recognizers between them.

### ModelStatus

Bases: `IntEnum`

Status of a model as reported by the server.

### ModelDependency

```
ModelDependency(id, role, status, size_bytes=0, folder='', local_path='', failure_reason='')
```

Editor-facing status of one shared-asset dependency of a parent model.

### TtsVoicePreset

```
TtsVoicePreset(id, label, description='')
```

Catalog-advertised speaker/description preset.

### TtsInlineTag

```
TtsInlineTag(token, kind='', verification='')
```

Model-native inline control token declared for authoring.

### TtsCapabilities

```
TtsCapabilities(voice_description=False, delivery_instruction=False, reference_voice=False, speed=False, language_per_call=False, voice_presets=(), inline_tags=())
```

Structured TTS capabilities advertised by the catalog.

### ModelInfo

```
ModelInfo(name, status, hf_repo, size_bytes, languages=(), dependencies=(), tts_capabilities=None)
```

Summary information for one model returned by `list_models()`.

### StringStorageKind

Bases: `IntEnum`

Kind of a StringStorage. Mirrors `StringStorageKind` in `messages.fbs`.

### CancelMode

Bases: `IntEnum`

How a mid-turn cancel treats the in-flight interaction. Mirrors `CancelMode` in `messages.fbs`.

### TurnStatus

Bases: `IntEnum`

Outcome of a workflow turn, as reported in `TurnComplete`. Mirrors `TurnStatus` in `messages.fbs`.

### NodeDesc

```
NodeDesc(name, params)
```

Internal serialisable description of a single node in the graph.

#### name

```
name
```

Graph-unique node name.

#### params

```
params
```

Typed node parameters. One concrete subclass per node type.

### GraphDescription

```
GraphDescription()
```

Fluent builder for a workflow graph sent to the server.

Typical usage::

```
graph = (GraphDescription()
         .add_generate("gen", GenerateParams(
             model_name="qwen2.5-0.5b-instruct",
             system_prompt="You are helpful."))
         .set_start_node("gen"))
agent = await client.create_agent(graph)
```

#### add_node

```
add_node(name, params)
```

Generic typed node add; accepts any concrete NodeParamsBase subclass.

#### set_start_node

```
set_start_node(name)
```

Set the entry-point node name.

#### set_default_model_name

```
set_default_model_name(name)
```

Set the fallback model name for nodes that need one.

______________________________________________________________________

## tryll_client.variables

`AgentProxy.variables` — a mapping-like typed store; see [Agent Variables](https://docs.tryllengine.com/v0.8/concepts/agent-variables.html) for the full model.

## tryll_client.variables

AgentVariables — per-agent typed variable mirror with deferred-flush staging.

Mirrors the store declared at `create_agent(variables=...)` time: a mapping- like local cache seeded from the declaration, pre-validated locally (unknown name / type mismatch raise immediately — no wire round-trip), with mutations coalesced per name and flushed as one batched `UpdateAgentVariablesRequest` immediately before the agent's next `send_message`/`send_message_async`/ `resume`/`change_param`/`change_params` call (never before `cancel`).

Staged writes are coalesced per name (last value wins for scalars; a minimal add/remove delta, or a whole-set assign, whichever is smaller, for sets) so rapid repeated writes never grow the batch unboundedly. A rejected flush is recovered rather than silently dropped: a transient `AgentBusy` re-stages the batch behind any newer writes; a hard rejection re-syncs the affected names from the server so the optimistic mirror never stays diverged. Either way the error propagates to the caller.

Same shape as the C++ client's `Tryll::Client::AgentVariables` — see `tryll/clients/cpp/include/tryll/AgentVariables.h`.

### VariableDecl

```
VariableDecl(value, allow_output_substitution=False)
```

One agent-variable value plus immutable declaration metadata.

### AgentVariables

```
AgentVariables(client, agent_id, declared=None)
```

Mapping-like local mirror of one agent's declared per-agent variables.

Seed the mirror from the declaration passed to `create_agent`.

Parameters:

| Name       | Type                                   | Description                                                                                       | Default    |
| ---------- | -------------------------------------- | ------------------------------------------------------------------------------------------------- | ---------- |
| `client`   | `'TryllClient'`                        | Parent :class:~tryll_client.client.TryllClient.                                                   | *required* |
| `agent_id` | `int`                                  | Server-assigned agent id this store belongs to.                                                   | *required* |
| `declared` | `Optional[dict[str, tuple[int, Any]]]` | {name: (value_type, initial_value)} — the exact declaration sent in CreateAgentRequest.variables. | `None`     |

#### snapshot

```
snapshot()
```

Return a deep copy of every declared variable's current local value.

#### set

```
set(name, value)
```

Stage an `Assign` — *value*'s inferred type must match the declaration.

A plain Python `int` is accepted for a variable declared `float` (matches the filter grammar's int-to-float promotion); every other type mismatch raises immediately, before anything is queued for the wire.

#### add_to_set

```
add_to_set(name, element)
```

Stage an idempotent `AddToSet` — *name* must be a declared string set.

#### remove_from_set

```
remove_from_set(name, element)
```

Stage an idempotent `RemoveFromSet` — *name* must be a declared string set.

#### reset

```
reset(name)
```

Stage a `Reset` — restores the `create_agent`-time initial value.

#### stage_batch

```
stage_batch(ops)
```

Stage a batch atomically: validate+stage every op, or roll back all.

*ops* is a list of `(op_name, name, value)` where `op_name` is one of `assign` / `add_to_set` / `remove_from_set` / `reset` (the `value` is ignored for `reset`). On the first invalid op the mirror and staged queue are restored to their pre-call state and the error is re-raised, so a rejected batch leaves nothing dirty — mirroring the server's own validate-all-then-apply atomicity.

#### flush_if_dirty

```
flush_if_dirty(timeout=10.0)
```

Send every staged mutation as one batched `UpdateAgentVariablesRequest`.

Called automatically at the top of `send_message`, `send_message_async`, `resume`, `change_param`, and `change_params` — deliberately *not* before `cancel`. No-op when nothing is staged.

The batch is retained in-flight until acknowledged. On a transient `AgentBusy` it is re-staged behind any newer writes and the error is re-raised; on a hard rejection the affected names are re-synced from the server before re-raising, so a rejected batch never leaves the mirror silently diverged.

#### sync_from_server

```
sync_from_server(names=None, timeout=10.0)
```

Refresh the local mirror from the server's authoritative current values.

Names with a locally newer (staged or in-flight) write are skipped, so a server snapshot never clobbers an unsent local change.

Parameters:

| Name    | Type                  | Description                                                         | Default |
| ------- | --------------------- | ------------------------------------------------------------------- | ------- |
| `names` | `Optional[list[str]]` | Variable names to fetch; empty/None fetches all declared variables. | `None`  |

### infer_variable_type

```
infer_variable_type(value)
```

Infer the VARIABLE_VALUE\_\* wire tag for a native Python value.

`bool` must be checked before `int` since `bool` is an `int` subclass in Python.

______________________________________________________________________

## tryll_client.kv_cache

`AgentProxy.prefill_kv_cache()`, `evict_kv_cache()`, and `get_kv_cache_status()` operate on the aggregate lifecycle state. Create an agent with `kv_cache_initialization=AgentKvCacheInitialization.Prefill` or `DeferAllocation` when needed. Sends restore evicted contexts automatically. See [Manage an Agent's KV Cache](https://docs.tryllengine.com/v0.8/how-to/manage-agent-kv-cache.html).

## tryll_client.kv_cache

KV-cache lifecycle types for server-side agents.

______________________________________________________________________

## tryll_client.memory

`TryllClient.get_memory_consumption()` returns a server-wide RAM/VRAM snapshot. Session-scoped (must follow `create_session`) but the numbers cover every session on the process. Never fails with `AgentBusy`. See [Inspect Server Memory](https://docs.tryllengine.com/v0.8/how-to/inspect-server-memory.html).

## tryll_client.memory

Memory-consumption snapshot types returned by GetMemoryConsumption.

### snapshot_from_fields

```
snapshot_from_fields(fields)
```

Build a public snapshot from a decoded GetMemoryConsumptionResponse fields dict.

______________________________________________________________________

## Dialog mutation

`AgentProxy.append_interactions()` and `AgentProxy.remove_interactions_from_end()` append or tail-remove scripted history without running the graph. Both are **strict idle-only** (`AgentBusy` 3004 while running, paused, or during a KV-cache lifecycle operation). Return applied/removed counts; no `TurnComplete`.

Each append item is `(user_message, assistant_message)` — `None` or empty strings omit that side; both empty → skipped. See [Seed and edit dialog history](https://docs.tryllengine.com/v0.8/how-to/seed-and-edit-dialog-history.html).

`export_dialog()` / `import_dialog()` capture or replace the completed dialog. `encode_flatbuffer` / `decode_flatbuffer` produce canonical `TDLG` bytes; the game owns storage. See [Export and import dialog](https://docs.tryllengine.com/v0.8/how-to/export-and-import-dialog.html) and [Save and load agent state](https://docs.tryllengine.com/v0.8/how-to/save-and-load-agent-state.html) for multi-agent envelopes, variables, and restore ordering.

______________________________________________________________________

## tryll_client.throttle

`TryllClient.set_inference_throttle(level)` reports how hard the server should yield the GPU back to your game — `0.0` = full speed (the default for a session that never calls it), `1.0` = maximum yielding. Fire-and-forget: no response, no error surfaced, safe to call every frame. It never changes what is generated, only how fast.

`create_agent(..., workload=AgentWorkload.Background)` marks an agent as background logic, so its inference gives way first under pressure. Everything else the server classifies from the graph itself, per turn. See [Wire Protocol](https://docs.tryllengine.com/v0.8/reference/wire-protocol.html#inference-throttle-and-agent-workload) and [Server configuration](https://docs.tryllengine.com/v0.8/reference/server-configuration.html#inference-scheduler).

## tryll_client.throttle

Manual inference throttle types for server-side agents and sessions.

### AgentWorkload

Bases: `IntEnum`

Whether a human ever waits on an agent's turns.

Purely a scheduling hint for the inference budget controller — it never changes node semantics. Set Background for world-logic/evaluator agents so their work is paced or deferred first under GPU pressure.

______________________________________________________________________

## tryll_client.errors

## tryll_client.errors

Exception type for Tryll client errors.

All failures surfaced by the synchronous client — server-reported errors, protocol-level decode failures, and timeouts — are raised as :class:`TryllError`. Server-reported errors carry a numeric `code` that matches the ranges defined in `server/common/include/tryll/ErrorCodes.h`; client-originated errors (timeouts, closed socket) carry `code == 0`.

### TryllError

```
TryllError(message, code=0)
```

Bases: `Exception`

Raised on server errors, protocol errors, or timeouts.

Attributes:

| Name   | Type | Description                                                                                                                                                                                                                                                 |
| ------ | ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `code` |      | Numeric error code from the server's ErrorCodes.h ranges (1xxx connection, 2xxx session, 3xxx agent, 4xxx model, 5xxx node, 6xxx graph, 7xxx string storage, 8xxx embedded storage). Zero for client-originated errors such as timeouts or a closed socket. |

Initialise a Tryll client error.

Parameters:

| Name      | Type  | Description                                                                                                                                                       | Default    |
| --------- | ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- |
| `message` | `str` | Human-readable description of the failure, typically copied from the server's ErrorResponse.message field or synthesised locally (e.g. "Timeout (30s) waiting…"). | *required* |
| `code`    | `int` | Numeric error code from ErrorCodes.h ranges; zero for client-originated errors.                                                                                   | `0`        |

#### __str__

```
__str__()
```

Return the message, prefixed with `[code]` when non-zero.

______________________________________________________________________

## tryll_client.managed_server

## tryll_client.managed_server

managed_server — spawn a tryll_server child process and wait for TCP readiness.

Typical usage::

```
from pathlib import Path
from tryll_client import TryllClient, ManagedServer

with ManagedServer.start(exe=Path("C:/tryll/tryll_server.exe"), port=9100) as srv:
    client = TryllClient.connect(srv.host, srv.port)
    client.create_session(...)
    # … use client …
    client.shutdown()
```

No executable discovery is performed here. Pass the path to the server executable explicitly (or use the helpers in `qa-and-eval/tryll_qa` to locate it).

### ServerProcessExitedError

```
ServerProcessExitedError(exit_code)
```

Bases: `RuntimeError`

Raised by :func:`wait_for_tcp` when the watched process exits before its TCP port ever opens.

*exit_code* is the process's exit code; *port_conflict* is `True` when it equals :data:`_SERVER_EXIT_PORT_IN_USE`, so callers can special-case a bind collision (retry on a fresh port) versus any other early exit (report the real failure instead of a generic timeout).

### ServerProcess

Bases: `NamedTuple`

Return value of :func:`start_server_process`.

*proc* is the spawned child. *port* is the port it was actually told to bind — when the caller left *port* at its default, :func:`start_server_process` auto-allocates a free ephemeral one, so this is the only way a direct caller can learn which port that turned out to be (the process was launched with `--port <port>`, not the caller's original default).

### ManagedServer

```
ManagedServer(*, exe, host='127.0.0.1', port=9100, cwd=None, extra_args=(), stdout=None, stderr=None, start_timeout=30.0, stop_timeout=8.0, idle_shutdown_timeout=_DEFAULT_IDLE_SHUTDOWN_TIMEOUT)
```

RAII handle around a child `tryll_server` process.

Use as a context manager (recommended), or call :meth:`stop` explicitly::

```
with ManagedServer.start(exe=Path("..."), port=9100) as srv:
    client = TryllClient.connect(srv.host, srv.port)
    ...
```

No executable discovery is performed — pass the path explicitly. The server is always launched with `--port <port>` so the caller's port takes precedence over whatever is in `server-config.json`, and (unless *idle_shutdown_timeout* is 0) with `--idle-shutdown-timeout <N>` so it self-exits once no session is using it — :meth:`stop` releases our handle but does not kill the process.

#### host

```
host
```

Host string to pass to :func:`TryllClient.connect`.

#### port

```
port
```

TCP port the server is listening on.

#### pid

```
pid
```

PID of the child process, or -1 if not running.

#### is_running

```
is_running
```

`True` while the child process is alive.

#### start

```
start(*, exe, host='127.0.0.1', port=9100, cwd=None, extra_args=(), stdout=None, stderr=None, start_timeout=30.0, stop_timeout=8.0, idle_shutdown_timeout=_DEFAULT_IDLE_SHUTDOWN_TIMEOUT)
```

Spawn the server and block until its TCP port is ready.

Parameters:

| Name                    | Type            | Description                                                                                                                                                                                                                                                                                                                                            | Default                                                           |
| ----------------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------- |
| `exe`                   | \`Path          | str\`                                                                                                                                                                                                                                                                                                                                                  | Path to tryll_server[.exe]. Required.                             |
| `host`                  | `str`           | Host string for the TCP ready-probe (default 127.0.0.1).                                                                                                                                                                                                                                                                                               | `'127.0.0.1'`                                                     |
| `port`                  | `int`           | TCP port; passed as --port to the server (default 9100).                                                                                                                                                                                                                                                                                               | `9100`                                                            |
| `cwd`                   | \`Path          | None\`                                                                                                                                                                                                                                                                                                                                                 | Working directory for the child process (defaults to exe.parent). |
| `extra_args`            | `Sequence[str]` | Additional CLI arguments appended after --port <port>.                                                                                                                                                                                                                                                                                                 | `()`                                                              |
| `stdout`                | \`Path          | None\`                                                                                                                                                                                                                                                                                                                                                 | File path for child stdout redirection (None = discard).          |
| `stderr`                | \`Path          | None\`                                                                                                                                                                                                                                                                                                                                                 | File path for child stderr redirection (None = discard).          |
| `start_timeout`         | `float`         | Seconds to wait for the TCP port to open (default 30).                                                                                                                                                                                                                                                                                                 | `30.0`                                                            |
| `stop_timeout`          | `float`         | Seconds to wait for clean exit on :meth:terminate (default 8).                                                                                                                                                                                                                                                                                         | `8.0`                                                             |
| `idle_shutdown_timeout` | `int`           | Seconds of idle (zero sessions, no in-flight downloads) after which the server self-exits; passed as --idle-shutdown-timeout <N> (default 60). :meth:stop does not kill the process — it relies on this timer — so pass 0 only when the caller has its own explicit reap strategy (e.g. the QA harness, which always calls :func:stop_server_process). | `_DEFAULT_IDLE_SHUTDOWN_TIMEOUT`                                  |

Returns:

| Name | Type              | Description                                              |
| ---- | ----------------- | -------------------------------------------------------- |
| `A`  | `'ManagedServer'` | class:ManagedServer whose port is accepting connections. |

Raises:

| Type                | Description                                     |
| ------------------- | ----------------------------------------------- |
| `FileNotFoundError` | If exe does not exist.                          |
| `TimeoutError`      | If the port does not open within start_timeout. |
| `OSError`           | If the process cannot be spawned.               |

#### stop

```
stop(*, timeout=None)
```

Release the process handle without killing the server.

The server self-exits via its idle-shutdown timer (managed mode). Callers that need an immediate, guaranteed stop (e.g. spawn succeeded but connect/handshake failed, so no session was ever created and nothing else can be relying on this process) should call :meth:`terminate` instead. The QA harness uses :func:`stop_server_process` directly for the same reason.

#### terminate

```
terminate(*, timeout=None)
```

Immediately hard-kill the child process, if still running.

Idempotent — safe to call more than once, including after :meth:`stop`. Intended for failure paths where the caller knows this process is theirs and nothing else can depend on it. Do not call this after a successful handshake, since another consumer could be relying on the server staying up until its own idle timeout.

### wait_for_tcp

```
wait_for_tcp(host, port, timeout_sec, poll_interval=0.25, proc=None)
```

Block until *host*:*port* accepts a TCP connection or the wait expires.

If *proc* is given, it is polled each iteration; a process that has already exited raises :class:`ServerProcessExitedError` immediately instead of waiting (e.g. a port collision, exit code 3 — see :data:`_SERVER_EXIT_PORT_IN_USE`). While the process is still alive the wait is extended to :data:`_ALIVE_HARD_CAP_SEC` (a slow-but-alive cold start is tolerated rather than treated as fatal); *timeout_sec* is the floor and, when no *proc* is given, the only bound.

Raises :class:`TimeoutError` if the port does not open before the effective cap and the process (if given) is still running.

### connection_ready_identity

```
connection_ready_identity(client)
```

Extract ConnectionReady identity fields from a connected `TryllClient`.

### wait_for_connection_ready

```
wait_for_connection_ready(host, port, timeout_sec, proc=None, connect_timeout=5.0, poll_interval=0.25)
```

Block until the server delivers `ConnectionReady` on *host*:*port*.

Stronger than :func:`wait_for_tcp`: readiness means the application-level `ConnectionReady` hello was actually received, not merely that the TCP port accepts. This closes the cold-start gap where the port accepts a connection well before the server has finished starting and can service it (the "ConnectionReady-late" failure mode) — a TCP-only check would declare "ready" prematurely.

Like :func:`wait_for_tcp`, it keeps trying while *proc* (if given) is alive, bounded by :data:`_ALIVE_HARD_CAP_SEC`, and short-circuits with :class:`ServerProcessExitedError` the instant the child exits. Raises :class:`TimeoutError` if the cap elapses while the child is still alive.

Returns the ConnectionReady identity dict (exe_path, build_config, version, run_id, managed, idle_timeout_seconds, protocol_version, codegen_fingerprint) captured from the successful probe connection.

### start_server_process

```
start_server_process(exe, port=_DEFAULT_PORT, cwd=None, env=None, log_dir=None, extra_args=(), host='127.0.0.1', idle_shutdown_timeout=_DEFAULT_IDLE_SHUTDOWN_TIMEOUT)
```

Start *exe* as a child process, passing `--port <port>` on the command line.

*cwd* defaults to the executable's directory (where `data/` usually lives).

*log_dir* is the directory where `server_stdout.txt` and `server_stderr.txt` are written. Defaults to the executable's directory. Pass `Path(os.devnull)` to suppress output entirely.

*extra_args* is appended to the command line after `--port <port>`; use it to pass server-startup flags such as `--disable-telemetry`.

When *port* is the default (9100) the function allocates a free ephemeral port automatically so every launcher gets its own isolated port. Pass an explicit port to override.

*idle_shutdown_timeout* is passed as `--idle-shutdown-timeout <N>` (default 60s) so the server self-exits once no session is using it — nothing calling this function kills the process on teardown. Pass 0 for a standalone server that should run until explicitly stopped (e.g. via :func:`stop_server_process`), or if *extra_args* already includes the flag.

Returns:

| Name | Type            | Description                                                    |
| ---- | --------------- | -------------------------------------------------------------- |
| `A`  | `ServerProcess` | class:ServerProcess carrying the child process and the port it |
|      | `ServerProcess` | was actually launched on — read .port rather than assuming the |
|      | `ServerProcess` | caller's port argument, since it may have been auto-allocated. |

### stop_server_process

```
stop_server_process(proc, *, terminate_timeout=8.0)
```

Terminate *proc*; `kill` if it does not exit within *terminate_timeout*.

### server_crash_info

```
server_crash_info(proc)
```

Classify whether *proc* crashed, based on whether it exited on its own.

The robust signal is "did the process exit *before* we asked it to stop":

- `poll()` is `None` — still running; the caller will `terminate()` it, which on Windows yields a nonzero exit code that must NOT be mistaken for a crash. Returns `(False, None)`.
- exited with code `0` — clean self-exit (unusual for the managed server). Returns `(False, 0)`.
- exited with a nonzero code — crashed (e.g. `0xC0000005` access violation). Returns `(True, code)`.

Call this *before* :func:`stop_server_process`.
