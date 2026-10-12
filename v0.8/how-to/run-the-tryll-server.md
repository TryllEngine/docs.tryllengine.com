# Run the Tryll Server

Start a Tryll server so clients can connect to it.

Prerequisites

- A Windows 10/11 machine (Linux support is evolving but not officially released).
- Either a pre-built `tryll_server.exe` from a release, or the source tree and a CMake + Visual Studio 2022 toolchain to build from source.

## Option A — use a pre-built server

If you have a packaged `tryll_server.exe` plus its `data/` folder:

1. Unpack the release into a directory, e.g. `C:\tryll`.

1. Open `data\server-config.json`. The defaults are fine for local development; only change what you need. See [Server Configuration](https://docs.tryllengine.com/v0.8/reference/server-configuration.html) for every field.

1. Confirm `data\models.json` lists at least one model you can use. If not, either add a local entry (see [Use Your Own Local Model](https://docs.tryllengine.com/v0.8/how-to/use-your-own-local-model.html)) or plan to download one after connecting.

1. Start the server:

   ```
   cd C:\tryll
   tryll_server.exe
   ```

   You should see a log line like `[Listener] Listening on 127.0.0.1:9100` (or whichever port you configured). The default `network_access: "local"` binds loopback (`127.0.0.1`), which never triggers a Windows Firewall prompt. Set `network_access: "local_network"` to bind all interfaces instead — the line then reads `[Listener] Listening on 0.0.0.0:9100 (private-network peers only)`.

1. In a new terminal, verify the port is open:

   ```
   netstat -an | findstr 9100
   ```

## Option B — build from source

If you want to hack on the server or your platform does not have a release yet:

1. Install prerequisites: CMake 3.24+, Visual Studio 2022 with the C++ workload, and Git.

1. Clone the repository and configure (build presets live in the `tryll/` tree, not `tryll/server/`):

   ```
   cd tryll
   cmake --preset default
   ```

   The `default` preset uses the Visual Studio 17 2022 generator with `TRYLL_CUDA=PREBUILT` and `TRYLL_VULKAN=PREBUILT` — both llama.cpp plugins, no GPU SDK. The committed `server-config.json` pins `backend` to `vulkan`. Prefer CUDA on NVIDIA with `--llama-backend auto` (or `cuda`). For a CPU-only build add `-DTRYLL_VULKAN=OFF -DTRYLL_CUDA=OFF`. The `production` preset ships the same llama.cpp CUDA+Vulkan payload with CPU ONNX Runtime (no CUDA Toolkit required). For ROCm see the internal developer docs. 3. Build:

   ```
   cmake --build --preset debug
   ```

1. Run:

   ```
   build\server\Debug\tryll_server.exe
   ```

   To point the server at a different config file:

   ```
   tryll_server.exe --config path\to\server-config.json
   ```

   To listen on a different port without editing the config file:

   ```
   tryll_server.exe --port 9200
   ```

   Both flags can be combined. `--port` overrides the `port` field from the config file. When using `ManagedServer` (C++ or Python clients) or the Unreal plugin auto-launch, the port is passed automatically — you do not need to edit `server-config.json`.

   Snapshot file writes (`SaveGameMemory` / `SaveDialog`) follow the process `saves` policy. An unmanaged operator sets `saves` in `server-config.json` (or `--save-root` / `--disable-saves` / `--allow-absolute-save-paths`). Auto-launched Unity/Unreal sidecars pass those flags for you. See [Server Configuration → Saves](https://docs.tryllengine.com/v0.8/reference/server-configuration.html#saves).

## What to edit in `server-config.json`

The fields you most often want to touch:

- **`mode`** — `tryll-dev`, `client-dev`, or (internal) `client-prod`. Prefer the CLI `--mode` flag. Debug/Release default `tryll-dev`; Production accepts only `client-dev`.
- **`network_access`** — `"local"` (default) binds loopback (`127.0.0.1`) and never prompts Windows Firewall; `"local_network"` binds all interfaces (`0.0.0.0`) but serves only private/LAN peers, so other machines on your network can connect.
- **`storage_root` / `models_root` / `logs_dir`** — process roots. Mode defaults supply them when omitted; the committed config leaves them unset on purpose. Prefer CLI `--storage-root`, `--models-root`, `--log-dir`.
- **Log files** — the deployment mode picks the file, not `log_mode`. Tryll Dev and Client Dev open a new `tryll_<timestamp>.log` per run; Client Prod rotates a single `tryll.log`. `log_mode` is still validated (`"production"` or `"dev"`).
- **`log_level`** — `"info"` is a fine default. `"trace"` turns on the per-turn workflow trace, which logs every node execution — noisy but great for debugging graphs.
- **`log_flush_on` / `log_flush_every_ms`** — how soon the log file catches up with the running process. Defaults (`"warn"` + 5000 ms) keep warnings immediate and everything else at most five seconds behind. Set `"off"` and `0` to leave buffering to the C library.
- **`default_canned_responses_path`** and **`default_guardrail_patterns_path`** — *server-admin fallback.* Point these at files to supply defaults for [CannedResponse](https://docs.tryllengine.com/v0.8/reference/nodes/canned-response.html) and [RegexGuardrail](https://docs.tryllengine.com/v0.8/reference/nodes/regex-guardrail.html) nodes **when the client has not created a string storage**. Most client workflows won't need these — use [`CreateStringStorageRequest`](https://docs.tryllengine.com/v0.8/reference/string-storage.html) (inline or `file_path`) instead.
- **`telemetry`** — behaviour depends on deployment mode. Client Dev always sends a usage lane (host + hardware + session started). Activity telemetry defaults on and is separately configurable. In Tryll Dev, telemetry is off unless `"enabled": true` (the shipped file sets it), and `"enabled": false` or `--disable-telemetry` turns everything off. See [Telemetry](https://docs.tryllengine.com/v0.8/reference/telemetry.html).

Every field is documented in [Server Configuration](https://docs.tryllengine.com/v0.8/reference/server-configuration.html).

## Verify it is working

In another terminal, try connecting with the C++ or Python client:

```
from tryll_client import TryllClient
c = TryllClient.connect("127.0.0.1", 9100)
c.create_session()  # required first (protocol v4)
print(c.list_models())
c.shutdown()
```

```
#include <tryll/TryllClient.h>
namespace TC = Tryll::Client;
auto client = TC::TryllClient::Connect("127.0.0.1", 9100);
client.CreateSession({}); // required first (protocol v4)
auto models = client.ListModels();
client.Shutdown();
```

The server log should show an accept, a `ConnectionReady`, a `CreateSessionRequest`, and a `ListModelsRequest`. (Model management — like every request other than `CreateSession` — requires a session first; calling `list_models()` before `CreateSession` returns `SessionNotReady`.)

## Graceful shutdown

`Ctrl+C` in the server console triggers a clean shutdown: the server cancels all active agent turns, closes all sessions, and exits. No client-visible data is kept across restarts — each session starts from scratch.

## Related

- [Reference: Server Configuration](https://docs.tryllengine.com/v0.8/reference/server-configuration.html)
- [Reference: Error Codes](https://docs.tryllengine.com/v0.8/reference/error-codes.html)
- [Concept: Architecture at a Glance](https://docs.tryllengine.com/v0.8/concepts/architecture-at-a-glance.html)
- [How-to: Connect and Manage a Session](https://docs.tryllengine.com/v0.8/how-to/connect-and-manage-session.html)
