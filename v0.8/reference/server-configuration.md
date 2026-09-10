# Server Configuration

The Tryll server reads two JSON files at startup: `server-config.json` (server settings — this page) and `models.json` (model catalog — see [model management](https://docs.tryllengine.com/v0.8/reference/model-management.html)). A third file, `downloads.json`, is written and maintained by the server itself at runtime and should not be edited by hand.

This page documents the user-facing fields of `server-config.json`. Relative paths inside the file are resolved against the directory containing the config file — **not** the process's working directory.

Built-in defaults vs the shipped file

The **Default** column below is the *built-in* value the server uses when a field is **absent**. The bundled `data/server-config.json` deliberately sets some fields differently for development — notably `logs_dir: ".app-data/logs"`, `models_download_dir: ".app-data/models"`, `log_mode: "dev"`, `log_level: "trace"`, `include_interaction_in_diagnostics: true`, and `engines.llama_cpp.backend: "vulkan"` (CUDA is opt-in via `--llama-backend` or the Unity/Unreal project setting). The release pipeline rewrites logging and diagnostics to quiet production values when it stages a shipped build; it leaves `backend` as `"vulkan"`. So the sample below reflects the shipped dev file, not the built-in defaults.

## Location and override

The default location is `data/server-config.json`, symlinked next to the server executable at build time. Two CLI flags can override values without editing the file:

```
tryll_server.exe --config path/to/server-config.json
tryll_server.exe --port 9200
tryll_server.exe --llama-backend vulkan
tryll_server.exe --network-access local_network
```

The flags can be combined. `--port` overrides the `port` field from the config file and is how `ManagedServer` (C++ and Python clients) ensures the port it connects to matches the port the server listens on. `--llama-backend` overrides `engines.llama_cpp.backend` — Unity and Unreal auto-launch pass this from Project Settings. `--network-access` overrides the `network_access` field.

Missing fields keep their defaults; the server runs with all defaults even if the file is absent.

## Minimal config

```
{
    "port": 9100,
    "thread_count": 4,
    "models_catalog_path": "models.json",
    "models_download_dir": ".app-data/models",
    "logs_dir": ".app-data/logs",
    "log_mode": "dev",
    "log_level": "trace"
}
```

## Fields

### Networking

| Field            | Type   | Default   | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ---------------- | ------ | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `port`           | uint16 | `9100`    | TCP port the server listens on. The [wire protocol](https://docs.tryllengine.com/v0.8/reference/wire-protocol.html) is plain TCP; clients connect to `localhost:<port>`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `network_access` | string | `"local"` | Which network interface the server binds. `"local"` (default) binds loopback (`127.0.0.1`) only — the game and the bundled server run on the same machine, nothing on the network can reach the server, and **Windows Firewall never prompts**. Use this for shipped single-machine builds. `"local_network"` binds all interfaces so other devices on your LAN can connect (e.g. game and server on different machines), but rejects any connection from outside the local network, so the public internet is never served. Note: `"local_network"` **will trigger the one-time Windows Firewall prompt** the first time the server runs, because it listens on a network interface. |
| `thread_count`   | uint   | `4`       | Size of the server's I/O thread pool. Increase for higher concurrency across many simultaneous sessions; does **not** affect single-agent inference throughput, which is bounded by the server's one-model-at-a-time inference queue.                                                                                                                                                                                                                                                                                                                                                                                                                                                 |

### Model catalog

| Field                 | Type   | Default              | Description                                                                                                                                                                                                                                                                          |
| --------------------- | ------ | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `models_catalog_path` | string | `"data/models.json"` | Path to the [model catalog](https://docs.tryllengine.com/v0.8/reference/glossary.html#model-catalog) JSON. Relative paths resolved from the config file's directory. A missing or invalid catalog aborts startup (non-zero exit); the server does not listen with an empty catalog.  |
| `models_download_dir` | string | `"data/.models/"`    | Directory where [HuggingFace](https://docs.tryllengine.com/v0.8/reference/glossary.html#gguf)-downloaded model files are stored. The server creates the directory if it does not exist. Also the location of `downloads.json`, the runtime-maintained record of completed downloads. |

### Logging

| Field                | Type   | Default        | Description                                                                                                                                                               |
| -------------------- | ------ | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `logs_dir`           | string | `"data/logs"`  | Directory for log files. Set to `""` (empty string) for console-only logging. The directory is created if it does not exist.                                              |
| `log_mode`           | string | `"production"` | Log file naming. `"production"` writes a single rotating `tryll.log` (10 MiB × 5 files). `"dev"` writes a new `tryll_<timestamp>.log` per process start with no rotation. |
| `log_level`          | string | `"info"`       | Minimum log level. One of `"trace"`, `"debug"`, `"info"`, `"warn"`, `"error"`. Use `"trace"` to capture per-node workflow events for debugging graphs.                    |
| `log_flush_on`       | string | `"warn"`       | Flush the log file as soon as a message at this level or higher is written. One of `"off"`, `"trace"`, `"debug"`, `"info"`, `"warn"`, `"error"`. `"off"` turns this off.  |
| `log_flush_every_ms` | int    | `5000`         | Also flush the log file on this interval so a quiet server does not leave the file minutes behind. `0` turns the timer off.                                               |

In both log modes the **console sink is always active**. By default the file is also flushed on every warning/error and every 5 seconds, so a running server's log is readable without waiting for process exit.

### Storage root

| Field          | Type   | Default              | Description                                                                                                                                                                                                                                                                                                       |
| -------------- | ------ | -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `storage_root` | string | `""` (exe directory) | **Server-wide default storage root.** Relative paths in node `string_storage` and `embedded_string_storage` params are resolved against this directory. Empty string means the directory that contains the `tryll_server` executable. Relative paths in this field are resolved from the config file's directory. |

Clients can override this per-session by sending a non-empty `storage_data_folder` in `CreateSessionRequest`; the per-session value takes precedence for the lifetime of that session.

Resolution order:

1. `CreateSessionRequest.storage_data_folder` (non-empty) — per-session override.
1. `storage_root` in `server-config.json` (non-empty) — server-wide default.
1. Directory containing the `tryll_server` executable — unconditional fallback.

Absolute paths and `..`-traversal that would escape the root are rejected with error `3009 StorageOutsideRoot`. Files that are inside the root but absent on disk produce `3010 StorageFileNotFound`. See [error codes](https://docs.tryllengine.com/v0.8/reference/error-codes.html#3xxx-agent) for recovery guidance.

### Node defaults

| Field                             | Type   | Default                                 | Description                                                                                                                                                                                                                                                                                                                                    |
| --------------------------------- | ------ | --------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `default_canned_responses_path`   | string | `"data/default-canned-responses.txt"`   | Path to the default response list for [canned-response nodes](https://docs.tryllengine.com/v0.8/reference/nodes/canned-response.html) that do not specify their own [string storage](https://docs.tryllengine.com/v0.8/reference/string-storage.html). UTF-8 text, one response per line; blank lines and lines starting with `#` are skipped. |
| `default_guardrail_patterns_path` | string | `"data/default-guardrail-patterns.txt"` | Path to the default regex list for [regex-guardrail nodes](https://docs.tryllengine.com/v0.8/reference/nodes/regex-guardrail.html) that do not specify their own string storage. Same format as above; patterns match case-insensitively.                                                                                                      |

Both files are loaded lazily on first use; an absent file on startup is fine as long as no node ever requests the default list.

### Diagnostics

| Field                                | Type | Default | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ------------------------------------ | ---- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `include_interaction_in_diagnostics` | bool | `false` | When `true`, **and** the client sets `enable_diagnostics = true` on `CreateAgentRequest`, the current [interaction](https://docs.tryllengine.com/v0.8/reference/glossary.html#interaction)'s components are serialised into `TurnDiagnostics.interaction` on `TurnComplete`. Each component appears as a flat JSON object with a `"_type"` discriminator — see [Turn Diagnostics JSON](https://docs.tryllengine.com/v0.8/reference/turn-diagnostics-json.html#interaction). Intended for debug sessions; payload grows linearly with component count. |
| `include_engine_diagnostics`         | bool | `false` | When `true`, **and** the client sets `enable_diagnostics = true`, per-node [`diagnostics.engine`](https://docs.tryllengine.com/v0.8/reference/turn-diagnostics-json.html#engine) (scheduler / backend metrics) is included in wire `debug_info`. Never contains prompt or generated text. Development configs typically enable it; production release staging turns it off.                                                                                                                                                                           |

### Telemetry

Tryll ships with usage telemetry **enabled by default**. The top-level `"telemetry"` object controls it. See [Telemetry](https://docs.tryllengine.com/v0.8/reference/telemetry.html) for the full background — what is collected, where it goes, and what you must do before shipping a game to players.

| Field                            | Type  | Default         | Description                                                                                                                                                         |
| -------------------------------- | ----- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `telemetry.enabled`              | bool  | `true`          | Master switch. Set to `false` to disable all telemetry with zero hot-path cost.                                                                                     |
| `telemetry.include_user_content` | bool  | `false`         | When `true`, human messages and character replies are included in turn events. Keep `false` unless you explicitly want conversation content in your analytics sink. |
| `telemetry.sinks`                | array | PostHog EU sink | List of sink objects. Remove or empty this array to stop sending data to PostHog (required before shipping a game to players).                                      |

Each sink in `"sinks"` supports:

| Field               | Type   | Description                                                                 |
| ------------------- | ------ | --------------------------------------------------------------------------- |
| `kind`              | string | Sink type. Currently only `"posthog"` is supported.                         |
| `enabled`           | bool   | Per-sink on/off switch.                                                     |
| `endpoint`          | string | PostHog ingest URL, e.g. `"https://eu.i.posthog.com"`.                      |
| `project_api_key`   | string | PostHog project API key (`phc_…`).                                          |
| `batch_max_records` | uint   | Maximum events per batch before flushing. Default `50`.                     |
| `batch_max_ms`      | uint   | Maximum milliseconds before a batch is flushed. Default `2000`.             |
| `queue_capacity`    | uint   | In-memory event queue size; oldest entry dropped when full. Default `5000`. |
| `shutdown_flush_ms` | uint   | How long (ms) to wait for queue drain on server shutdown. Default `5000`.   |

### Speech (STT)

| Field                | Type   | Default        | Description                                                                                                                |
| -------------------- | ------ | -------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `default_vad_model`  | string | `"Silero VAD"` | Catalog name of the voice-activity-detection model the STT engine loads for hands-free / endpointed capture.               |
| `stt_debug_dump_dir` | string | `""`           | When non-empty, the STT engine writes one WAV per VAD segment under this directory. Debug-only; leave empty in production. |

### Process lifetime

| Field                   | Type          | Default | Description                                                                                                                                                                                                                                           |
| ----------------------- | ------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `idle_shutdown_timeout` | int (seconds) | `0`     | Seconds of idle (zero sessions, no in-flight downloads) before a **managed** server self-exits. `0` = never exit (standalone). CLI override: `--idle-shutdown-timeout <sec>`. Auto-launching clients set this so the server shuts down with the game. |

### Dev-time monitor (`monitor`)

An optional HTTP/SSE sidecar for inspecting live turns during development. Only present in non-Production builds and gated by `monitor.enabled`.

| Field                             | Type   | Default       | Description                                                                                                                            |
| --------------------------------- | ------ | ------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `monitor.enabled`                 | bool   | `false`       | Turn the monitor sidecar on.                                                                                                           |
| `monitor.bind`                    | string | `"127.0.0.1"` | Interface the monitor HTTP server binds.                                                                                               |
| `monitor.port`                    | uint16 | `9101`        | Monitor HTTP port.                                                                                                                     |
| `monitor.turn_ring_size`          | uint   | `200`         | How many recent turns to retain in the ring buffer.                                                                                    |
| `monitor.memory_tick_interval_ms` | uint   | `0`           | Interval in milliseconds for the monitor's live memory sample. `0` disables periodic sampling; the Memory panel still loads on demand. |
| `monitor.allow_remote`            | bool   | `false`       | Allow non-loopback clients to reach the monitor.                                                                                       |

### Crash dumps (`crash_dump`)

| Field                | Type   | Default | Description                                                                                        |
| -------------------- | ------ | ------- | -------------------------------------------------------------------------------------------------- |
| `crash_dump.enabled` | bool   | `true`  | Write a minidump on a fatal crash. Ships in **all** configs, including Production.                 |
| `crash_dump.dir`     | string | `""`    | Where minidumps land. Empty → `logs_dir` → exe directory. CLI override: `--crash-dump-dir <path>`. |

### Debug commands (`debug_commands`)

| Field                    | Type | Default | Description                                                                                                                             |
| ------------------------ | ---- | ------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `debug_commands.enabled` | bool | `false` | Enable the dev-only `DebugCommandRequest` channel. Compiled out of Production builds entirely. CLI override: `--enable-debug-commands`. |

### Engine tuning (`engines`)

Per-engine tuning, keyed by backend. All fields are optional and default to the prior hardcoded values; most projects never touch this block.

```
"engines": {
    "llama_cpp": {
        "backend": "cuda",
        "n_threads": 4,
        "n_threads_batch": 4,
        "inference": {
            "default_n_ctx": 8192,
            "n_batch": 512,
            "offload_kqv": true,
            "scheduler": {
                "enabled": true,
                "prefill_yield_enabled": true,
                "prefill_quantum_tokens": 512,
                "interactive_visible":  { "prefill_quantum_tokens": 256, "contention_scaling": true,  "max_pause_ms": 10 },
                "interactive_buffered": { "prefill_quantum_tokens": 512, "contention_scaling": false, "max_pause_ms": 50 },
                "background":           { "prefill_quantum_tokens": 128, "contention_scaling": true,  "max_pause_ms": 50 }
            }
        },
        "embedding": { "default_n_ctx": 512 }
    }
}
```

`engines.llama_cpp.backend` (`auto` | `cuda` | `vulkan` | `hip` | `cpu`) selects the ggml GPU plugin. The committed file and shipped plugin default to `"vulkan"`. `auto` prefers CUDA on NVIDIA when that DLL loads, otherwise Vulkan, otherwise CPU. Override per process with `--llama-backend`. Unity and Unreal auto-launch pass that flag from Project Settings instead of editing this file. This is independent of per-model RAM/VRAM placement. The payload ships both llama.cpp plugins plus the llama-cuda redistributables (`cudart` / `cublas`). It does not ship cuDNN or a CUDA ONNX Runtime.

GPU offload moved

`engines.llama_cpp.n_gpu_layers` is **deprecated and ignored**. It applied one value to every model in the process, which cannot be right once different models want different memory. Offload is now decided per model from its placement — see [Choose where models run](https://docs.tryllengine.com/v0.8/how-to/choose-where-models-run.html). A server that still finds the key logs a warning once at startup; remove it.

`default_n_ctx` is the fallback context window when neither a node's `context_size` nor the model variant's `context_size` is set.

#### Inference scheduler

The cooperative scheduler splits long prompt processing into bounded chunks so several agents can share one model without one long prompt starving the rest, and implements the [manual inference throttle](https://docs.tryllengine.com/v0.8/reference/wire-protocol.html#inference-throttle-and-agent-workload).

| Field                    | Type | Default | Description                                                     |
| ------------------------ | ---- | ------- | --------------------------------------------------------------- |
| `enabled`                | bool | `true`  | Master switch for cooperative scheduling.                       |
| `prefill_yield_enabled`  | bool | `true`  | Chunk prompt processing so other agents can interleave.         |
| `prefill_quantum_tokens` | int  | `512`   | Legacy global chunk size, used when no per-class block applies. |

The three per-class blocks — `interactive_visible`, `interactive_buffered` and `background` — tune each [consumer class](https://docs.tryllengine.com/v0.8/reference/wire-protocol.html#createagentrequestworkload-declaring-background-agents) separately. Any block you omit falls back to built-in defaults; a partial block keeps defaults for the fields you leave out.

| Field                    | Type | Description                                                                                                                                                                                                                                                                   |
| ------------------------ | ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `prefill_quantum_tokens` | int  | Prompt-processing chunk size for this class. Clamped at runtime to the device's `n_batch`, which always wins.                                                                                                                                                                 |
| `contention_scaling`     | bool | Divide the chunk size by the number of prompts competing for this model, so a burst of agents interleaves more finely. Not shrunk below 64 tokens (below that, fixed per-chunk overhead dominates) unless `n_batch` or your own value is smaller.                             |
| `max_pause_ms`           | int  | Longest pause inserted between generated tokens at throttle `1`. Scales linearly with the reported level, so `0.3` on a 50 ms cap pauses 15 ms. Also bounds how long a "stop talking" cancel can take to land, which is why it is a small fixed cap rather than a multiplier. |

Only `interactive_buffered`'s 512-token chunk is measurement-backed; the rest are reasonable starting points. Two guidelines behind the shipped defaults:

- **Visible text gets the smallest pause (10 ms).** A gap in text a player is reading is noticed immediately, so there is little slack to spend.
- **Spoken and background work get 50 ms.** A spoken turn's LLM is already blocked on speech synthesis for most of its duration, so pausing inside that slack costs nothing audible; background work has nobody waiting at all.

Raising `max_pause_ms` yields more GPU per throttle step but makes cancellation less prompt. If a throttled agent feels sluggish to interrupt, lower it.

## Log-mode details

| Value          | File naming                         | Rotation                      |
| -------------- | ----------------------------------- | ----------------------------- |
| `"production"` | `tryll.log`                         | 10 MiB max, 5 files kept      |
| `"dev"`        | `tryll_2026-04-06_14-30-00_123.log` | No rotation; new file per run |

## Related

- [Telemetry](https://docs.tryllengine.com/v0.8/reference/telemetry.html) — what is collected, where it goes, and how to disable before shipping.
- [Model Management](https://docs.tryllengine.com/v0.8/reference/model-management.html) — `models.json` schema and lifecycle.
- [How to run the Tryll server](https://docs.tryllengine.com/v0.8/how-to/run-the-tryll-server.html) — putting the config into practice.
- [Error codes](https://docs.tryllengine.com/v0.8/reference/error-codes.html) — what happens when a config-driven path is missing or malformed.
- [Glossary](https://docs.tryllengine.com/v0.8/reference/glossary.html)
