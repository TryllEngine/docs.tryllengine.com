# Server Configuration

The Tryll server reads two JSON files at startup: `server-config.json` (server settings — this page) and `models.json` (model catalog — see [model management](https://docs.tryllengine.com/v0.9/reference/model-management.html)). A third file, `downloads.json`, is written and maintained by the server itself at runtime and should not be edited by hand.

This page documents the user-facing fields of `server-config.json`. Relative paths inside the file are resolved against the directory containing the config file — **not** the process's working directory.

Built-in defaults vs the shipped file

The **Default** column below is the *built-in* value the server uses when a field is **absent**. Mode defaults supply `logs_dir` and `models_root` when those keys are omitted — the committed `data/server-config.json` deliberately sets **no** root keys so every mode gets the right layout. Development configs typically set `log_mode: "dev"`, `log_level: "trace"`, `include_interaction_in_diagnostics: true`, and `engines.llama_cpp.backend: "vulkan"` (CUDA is opt-in via `--llama-backend` or the Unity/Unreal project setting). See deployment modes in the contributor docs for per-mode roots and telemetry lanes.

## Location and override

The default location is `data/server-config.json`, symlinked next to the server executable at build time. Two CLI flags can override values without editing the file:

```
tryll_server.exe --config path/to/server-config.json
tryll_server.exe --port 9200
tryll_server.exe --mode client-dev
tryll_server.exe --project-name MyGame --project-dir C:\proj\MyGame
tryll_server.exe --storage-root C:\proj\MyGame\TryllStorage
tryll_server.exe --models-root C:\proj\MyGame\TryllStorage\models
tryll_server.exe --log-dir C:\Users\...\AppData\Local\Tryll\projects\...\logs
tryll_server.exe --llama-backend vulkan
tryll_server.exe --network-access local_network
```

The flags can be combined. `--port` overrides the `port` field from the config file and is how `ManagedServer` (C++ and Python clients) ensures the port it connects to matches the port the server listens on. `--llama-backend` overrides `engines.llama_cpp.backend` — Unity and Unreal auto-launch pass this from Project Settings. `--network-access` overrides the `network_access` field. Root precedence is CLI → config → mode default.

Missing fields keep their defaults. In Tryll Dev the server also runs with all defaults when the file is absent. Client Dev and Client Prod refuse to start without the file, because they will not guess their folders.

## Minimal config

```
{
    "port": 9100,
    "thread_count": 4,
    "models_catalog_path": "models.json",
    "log_mode": "dev",
    "log_level": "trace"
}
```

## Fields

### Deployment mode

| Field  | Type   | Default         | Description                                                                                                                                                                           |
| ------ | ------ | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `mode` | string | build-dependent | `tryll-dev`, `client-dev`, `client-prod`, or (later) `standalone`. Debug/Release default `tryll-dev` and accept the first three. Production accepts only `client-dev`. CLI: `--mode`. |

### Networking

| Field            | Type   | Default   | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ---------------- | ------ | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `port`           | uint16 | `9100`    | TCP port the server listens on. The [wire protocol](https://docs.tryllengine.com/v0.9/reference/wire-protocol.html) is plain TCP; clients connect to `localhost:<port>`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `network_access` | string | `"local"` | Which network interface the server binds. `"local"` (default) binds loopback (`127.0.0.1`) only — the game and the bundled server run on the same machine, nothing on the network can reach the server, and **Windows Firewall never prompts**. Use this for shipped single-machine builds. `"local_network"` binds all interfaces so other devices on your LAN can connect (e.g. game and server on different machines), but rejects any connection from outside the local network, so the public internet is never served. Note: `"local_network"` **will trigger the one-time Windows Firewall prompt** the first time the server runs, because it listens on a network interface. |
| `thread_count`   | uint   | `4`       | Size of the server's I/O thread pool. Increase for higher concurrency across many simultaneous sessions; does **not** affect single-agent inference throughput, which is bounded by the server's one-model-at-a-time inference queue.                                                                                                                                                                                                                                                                                                                                                                                                                                                 |

### Model catalog

| Field                      | Type   | Default                                        | Description                                                                                                                                                                                                                                                                                                                                              |
| -------------------------- | ------ | ---------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `models_catalog_path`      | string | `"data/models.json"`                           | Path to the shipped [model catalog](https://docs.tryllengine.com/v0.9/reference/glossary.html#model-catalog) JSON. Relative paths resolved from the config file's directory. A missing or invalid **base** catalog aborts startup (non-zero exit).                                                                                                       |
| `user_models_catalog_path` | string | sibling `user_models.json` of the base catalog | Optional user catalog. Nothing is shipped at the default path. Omitted field resolves next to the resolved base catalog (not `data/data/user_models.json`). Missing file is silent. CLI: `--user-models-path` (absolute, or relative to the process cwd). Unity/Unreal set this from an optional project-relative path in Project Settings or from code. |
| `user_models_catalog_dir`  | string | sibling `models.d` of the base catalog         | Directory of optional extra user `*.json` catalogs. The install does not create it. Omitted field resolves next to the resolved base catalog (not `data/data/models.d`). Missing directory is silent. CLI: `--user-models-dir`.                                                                                                                          |
| `user_catalog_on_error`    | string | `"skip"`                                       | `"skip"` logs and ignores broken user files; `"fail"` aborts after reporting all issues (CI). `--check-catalogs` always fails on any issue.                                                                                                                                                                                                              |
| `models_root`              | string | mode default                                   | Directory for downloaded model files and `downloads.json`. Replaces removed `models_download_dir` (no alias). Tryll Dev default: `data/.models/`. Client modes: `<storage_root>/models/` (must stay inside the storage root). CLI: `--models-root`.                                                                                                      |

### Logging

| Field                | Type   | Default        | Description                                                                                                                                                                                    |
| -------------------- | ------ | -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `logs_dir`           | string | mode default   | Directory for log files **and crash dumps**. Client Dev: Tryll app data under `projects/<key>/logs`. Client Prod: required; refuses to start if missing or unwritable. CLI: `--log-dir`.       |
| `log_mode`           | string | `"production"` | Must be `"production"` or `"dev"`. The value is validated and stored, but it does not choose the log file: file naming follows the deployment mode (see [Log file naming](#log-mode-details)). |
| `log_level`          | string | `"info"`       | Minimum log level. One of `"trace"`, `"debug"`, `"info"`, `"warn"`, `"error"`. Use `"trace"` to capture per-node workflow events for debugging graphs.                                         |
| `log_flush_on`       | string | `"warn"`       | Flush the log file as soon as a message at this level or higher is written. One of `"off"`, `"trace"`, `"debug"`, `"info"`, `"warn"`, `"error"`. `"off"` turns this off.                       |
| `log_flush_every_ms` | int    | `5000`         | Also flush the log file on this interval so a quiet server does not leave the file minutes behind. `0` turns the timer off.                                                                    |

In every mode the **console sink is always active**. By default the file is also flushed on every warning/error and every 5 seconds, so a running server's log is readable without waiting for process exit.

### Storage root

| Field          | Type   | Default      | Description                                                                                                                                                                                                                                                                        |
| -------------- | ------ | ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `storage_root` | string | mode default | **Process-wide storage root** fixed at startup. Relative paths in node `string_storage` and `embedded_string_storage` params resolve against it. Protocol v24 removed per-session `storage_data_folder`. Client Dev default: `<project-dir>/TryllStorage/`. CLI: `--storage-root`. |

Resolution order: CLI `--storage-root` → config `storage_root` → mode default.

Absolute paths and `..`-traversal that would escape the root are rejected with error `3009 StorageOutsideRoot`. Files that are inside the root but absent on disk produce `3010 StorageFileNotFound`. See [error codes](https://docs.tryllengine.com/v0.9/reference/error-codes.html#3xxx-agent) for recovery guidance.

Authored node / KB / voice paths stay on this root. Snapshot file APIs (`SaveGameMemory`, `LoadGameMemory`, `SaveDialog`, `LoadDialog`) use the [Saves](#saves) policy below. Unity/Unreal: see [Ship Storage Folders for Builds](https://docs.tryllengine.com/v0.9/how-to/ship-storage-folders-for-builds.html).

### Saves

Process-owned policy for snapshot files. The client still sends a `path` on each request; the **server process** decides whether writes are allowed, which folder relative paths join to, and whether absolute paths are accepted. There is no `CreateSession` override — a standalone server must not take a write root from the client.

| Field                        | Type   | Default | Description                                                                                                                                                                                                                                                                       |
| ---------------------------- | ------ | ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `saves.enabled`              | bool   | `true`  | When `false`, `SaveGameMemory` and `SaveDialog` are rejected with `8109 SavesDisabled`. `Load*` still resolve and read.                                                                                                                                                           |
| `saves.root`                 | string | `""`    | Directory relative save paths join to. Empty keeps today's rule: the process storage root. A relative value resolves against the config file's directory. When `enabled` and `root` are both set, the server creates the directory at startup and refuses to start if that fails. |
| `saves.allow_absolute_paths` | bool   | `false` | When `true`, an absolute `path` is used as-is. Client Prod / Standalone reject absolute request paths. Unity/Unreal auto-launch pass `--allow-absolute-save-paths` in the editor.                                                                                                 |

CLI overrides (same priority as `--port`: CLI wins):

- `--disable-saves`
- `--save-root <absolute>` (must be absolute; not re-resolved against the config file)
- `--allow-absolute-save-paths`

Resolution of a save/load `path`:

1. Empty `path` is rejected (`3009`).
1. A **write** with `saves.enabled` false is `8109`.
1. An **absolute** `path` is accepted only when `allow_absolute_paths` is true; otherwise `3009`.
1. A **relative** `path` joins `saves.root` when set, otherwise the process storage root. `..` that escapes that root is `3009`.

Unity/Unreal sidecars: Project Settings **Save Data Folder**. Empty in the editor omits `--save-root` (Knowledge / relative saves stay under TryllStorage). Empty in a player build passes `Application.persistentDataPath` (Unity) or `ProjectSavedDir()` (Unreal).

### Node defaults

| Field                             | Type   | Default                                 | Description                                                                                                                                                                                                                                                                                                                                    |
| --------------------------------- | ------ | --------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `default_canned_responses_path`   | string | `"data/default-canned-responses.txt"`   | Path to the default response list for [canned-response nodes](https://docs.tryllengine.com/v0.9/reference/nodes/canned-response.html) that do not specify their own [string storage](https://docs.tryllengine.com/v0.9/reference/string-storage.html). UTF-8 text, one response per line; blank lines and lines starting with `#` are skipped. |
| `default_guardrail_patterns_path` | string | `"data/default-guardrail-patterns.txt"` | Path to the default regex list for [regex-guardrail nodes](https://docs.tryllengine.com/v0.9/reference/nodes/regex-guardrail.html) that do not specify their own string storage. Same format as above; patterns match case-insensitively.                                                                                                      |

Both files are loaded lazily on first use; an absent file on startup is fine as long as no node ever requests the default list.

### Workflow (`workflow`)

| Field                                 | Type     | Default                      | Description                                                                                                                                                                                                                                            |
| ------------------------------------- | -------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `workflow.tool_call_pause_timeout_ms` | int (ms) | `0` (shipped config: `2000`) | Pause budget for [ToolCall](https://docs.tryllengine.com/v0.9/reference/nodes/tool-call.html) nodes whose own `pause_timeout_ms` is `0`. Resolved once at agent creation. On expiry the turn ends with `TurnStatus.PauseTimedOut`. `0` = wait forever. |
| `workflow.pause_stall_warn_ms`        | int (ms) | `15000`                      | How long a paused turn may wait before the server logs a one-shot warning naming the paused node. Logging only; it never ends the pause.                                                                                                               |

### Diagnostics

| Field                                | Type | Default | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ------------------------------------ | ---- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `include_interaction_in_diagnostics` | bool | `false` | When `true`, **and** the client sets `enable_diagnostics = true` on `CreateAgentRequest`, the current [interaction](https://docs.tryllengine.com/v0.9/reference/glossary.html#interaction)'s components are serialised into `TurnDiagnostics.interaction` on `TurnComplete`. Each component appears as a flat JSON object with a `"_type"` discriminator — see [Turn Diagnostics JSON](https://docs.tryllengine.com/v0.9/reference/turn-diagnostics-json.html#interaction). Intended for debug sessions; payload grows linearly with component count. |
| `include_engine_diagnostics`         | bool | `false` | When `true`, **and** the client sets `enable_diagnostics = true`, per-node [`diagnostics.engine`](https://docs.tryllengine.com/v0.9/reference/turn-diagnostics-json.html#engine) (scheduler / backend metrics) is included in wire `debug_info`. Never contains prompt or generated text. Development configs typically enable it; production release staging turns it off.                                                                                                                                                                           |

### Telemetry

Telemetry behaviour depends on **deployment mode**. In Client Dev, a **usage** lane (host ID, hardware, session started) is always on. An **activity** lane (everything after session start) defaults on and is separately configurable. Tryll Dev sends nothing unless `enabled` is `true`, and `--disable-telemetry` turns everything off. Client Prod and Standalone emit nothing. Events are sent only when `sinks` lists an enabled sink. See [Telemetry](https://docs.tryllengine.com/v0.9/reference/telemetry.html).

The shipped `data/server-config.json` sets `enabled: true` and lists one PostHog EU sink. The defaults below are what the server uses when a field is absent.

| Field                            | Type  | Default | Description                                                                                                                                                  |
| -------------------------------- | ----- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `telemetry.enabled`              | bool  | `false` | In Tryll Dev: master switch (false disables all lanes and host-ID capture). In Client Dev: usage telemetry stays on even when false (server logs a warning). |
| `telemetry.activity_enabled`     | bool  | `true`  | Gates every event after session start. Independent of the mandatory Client Dev usage lane.                                                                   |
| `telemetry.include_user_content` | bool  | `true`  | When `true`, slot text and tool-call arguments are included in **activity** turn events. Never sent on the usage lane.                                       |
| `telemetry.sinks`                | array | `[]`    | List of sink objects. With no active sink the server logs that telemetry is enabled but has nowhere to send it, and emits nothing.                           |

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

| Field                | Type   | Default        | Description                                                                                                                                                        |
| -------------------- | ------ | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `default_vad_model`  | string | `"Silero VAD"` | Catalog name of the voice-activity-detection model the STT engine loads for hands-free / endpointed capture.                                                       |
| `stt_debug_dump_dir` | string | `""`           | When non-empty, the STT engine writes one WAV per VAD segment under this directory. Debug-only and Tryll Dev only: Client Dev and Client Prod clear it at startup. |

### Process lifetime

| Field                   | Type          | Default | Description                                                                                                                                                                                                                                                                                                                                           |
| ----------------------- | ------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `idle_shutdown_timeout` | int (seconds) | `0`     | Seconds of idle (zero sessions, no in-flight downloads) before a **managed** server self-exits. `0` = never exit (**unmanaged**). CLI override: `--idle-shutdown-timeout <sec>`. Auto-launching clients set this so the server shuts down with the game. Do not call this lifetime “standalone”; that word names only the Standalone deployment mode. |

### Dev-time monitor (`monitor`)

An optional HTTP/SSE sidecar for inspecting live turns during development. Only present in non-Production builds, and only in Tryll Dev: Client Dev and Client Prod turn it off at startup whatever the config says, so the server that Unity and Unreal auto-launch (`client-dev`) never runs it.

| Field                             | Type   | Default       | Description                                                                                                                            |
| --------------------------------- | ------ | ------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `monitor.enabled`                 | bool   | `false`       | Turn the monitor sidecar on. Tryll Dev only.                                                                                           |
| `monitor.bind`                    | string | `"127.0.0.1"` | Interface the monitor HTTP server binds.                                                                                               |
| `monitor.port`                    | uint16 | `9101`        | Monitor HTTP port.                                                                                                                     |
| `monitor.turn_ring_size`          | uint   | `200`         | How many recent turns to retain in the ring buffer.                                                                                    |
| `monitor.memory_tick_interval_ms` | uint   | `0`           | Interval in milliseconds for the monitor's live memory sample. `0` disables periodic sampling; the Memory panel still loads on demand. |
| `monitor.allow_remote`            | bool   | `false`       | Allow non-loopback clients to reach the monitor.                                                                                       |

### Crash dumps (`crash_dump`)

| Field                | Type | Default | Description                                                                        |
| -------------------- | ---- | ------- | ---------------------------------------------------------------------------------- |
| `crash_dump.enabled` | bool | `true`  | Write a minidump on a fatal crash. Ships in **all** configs, including Production. |

Crash dumps always go to the logs folder (`logs_dir` / `--log-dir`). The old `crash_dump.dir` key and `--crash-dump-dir` flag are removed with no aliases.

### Debug commands (`debug_commands`)

| Field                    | Type | Default | Description                                                                                                                                                                                                                                       |
| ------------------------ | ---- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `debug_commands.enabled` | bool | `false` | Enable the dev-only `DebugCommandRequest` channel. Compiled out of Production builds entirely. CLI override: `--enable-debug-commands`. The key and the flag apply in Tryll Dev only; Client Dev and Client Prod turn the channel off at startup. |

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

`engines.llama_cpp.n_gpu_layers` is **deprecated and ignored**. It applied one value to every model in the process, which cannot be right once different models want different memory. Offload is now decided per model from its placement — see [Choose where models run](https://docs.tryllengine.com/v0.9/how-to/choose-where-models-run.html). A server that still finds the key logs a warning once at startup; remove it.

`default_n_ctx` is the fallback context window when neither a node's `context_size` nor the model variant's `context_size` is set.

#### Inference scheduler

The cooperative scheduler splits long prompt processing into bounded chunks so several agents can share one model without one long prompt starving the rest, and implements the [manual inference throttle](https://docs.tryllengine.com/v0.9/reference/wire-protocol.html#inference-throttle-and-agent-workload).

| Field                    | Type | Default | Description                                                     |
| ------------------------ | ---- | ------- | --------------------------------------------------------------- |
| `enabled`                | bool | `true`  | Master switch for cooperative scheduling.                       |
| `prefill_yield_enabled`  | bool | `true`  | Chunk prompt processing so other agents can interleave.         |
| `prefill_quantum_tokens` | int  | `512`   | Legacy global chunk size, used when no per-class block applies. |

The three per-class blocks — `interactive_visible`, `interactive_buffered` and `background` — tune each [consumer class](https://docs.tryllengine.com/v0.9/reference/wire-protocol.html#createagentrequestworkload-declaring-background-agents) separately. Any block you omit falls back to built-in defaults; a partial block keeps defaults for the fields you leave out.

| Field                    | Type | Description                                                                                                                                                                                                                                                                   |
| ------------------------ | ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `prefill_quantum_tokens` | int  | Prompt-processing chunk size for this class. Clamped at runtime to the device's `n_batch`, which always wins.                                                                                                                                                                 |
| `contention_scaling`     | bool | Divide the chunk size by the number of prompts competing for this model, so a burst of agents interleaves more finely. Not shrunk below 64 tokens (below that, fixed per-chunk overhead dominates) unless `n_batch` or your own value is smaller.                             |
| `max_pause_ms`           | int  | Longest pause inserted between generated tokens at throttle `1`. Scales linearly with the reported level, so `0.3` on a 50 ms cap pauses 15 ms. Also bounds how long a "stop talking" cancel can take to land, which is why it is a small fixed cap rather than a multiplier. |

Only `interactive_buffered`'s 512-token chunk is measurement-backed; the rest are reasonable starting points. Two guidelines behind the shipped defaults:

- **Visible text gets the smallest pause (10 ms).** A gap in text a player is reading is noticed immediately, so there is little slack to spend.
- **Spoken and background work get 50 ms.** A spoken turn's LLM is already blocked on speech synthesis for most of its duration, so pausing inside that slack costs nothing audible; background work has nobody waiting at all.

Raising `max_pause_ms` yields more GPU per throttle step but makes cancellation less prompt. If a throttled agent feels sluggish to interrupt, lower it.

## Log file naming

The deployment mode decides the log file, whatever `log_mode` says.

| Mode                  | File naming                         | Rotation                      |
| --------------------- | ----------------------------------- | ----------------------------- |
| Tryll Dev, Client Dev | `tryll_2026-04-06_14-30-00_123.log` | No rotation; new file per run |
| Client Prod           | `tryll.log`                         | 10 MiB max, 5 files kept      |

## Related

- [Telemetry](https://docs.tryllengine.com/v0.9/reference/telemetry.html) — what is collected, where it goes, and how to disable before shipping.
- [Model Management](https://docs.tryllengine.com/v0.9/reference/model-management.html) — `models.json` schema and lifecycle.
- [How to run the Tryll server](https://docs.tryllengine.com/v0.9/how-to/run-the-tryll-server.html) — putting the config into practice.
- [Error codes](https://docs.tryllengine.com/v0.9/reference/error-codes.html) — what happens when a config-driven path is missing or malformed.
- [Glossary](https://docs.tryllengine.com/v0.9/reference/glossary.html)
