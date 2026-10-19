# Auto-launch the Server

Let your client application start the Tryll server automatically instead of requiring a separately-launched process.

```
flowchart LR
    ManagedServer -->|"--port N"| ServerProcess["tryll_server"]
    ServerProcess -->|"TCP ready"| Client
    Client -->|"Connect(host, port)"| Session
```

The client spawns the server with `--port <N>` on the command line so the port both sides use is always in sync — no editing of `server-config.json` needed. Unity and Unreal also pass `--llama-backend vulkan|auto` from **Project Settings → Llama.cpp backend** (default Vulkan). That setting is ignored when Auto Launch Server is off: you then own the process and the config file.

______________________________________________________________________

## Unity

The Unity plugin ships the server **inside the package** and auto-launches it — no path configuration required. The server binaries live in the package's `.Server/` folder (`Packages/com.tryll.client/.Server/`):

- **Editor Play Mode** launches `tryll_server.exe` with `--mode client-dev`, `--project-name`, `--project-dir`, and `--storage-root` (default `<Project>/TryllStorage/`).
- **Player builds** copy the server payload into `<Build>/Server/` and stage `TryllStorage/` into `StreamingAssets/Tryll/`. Until a Client Prod binary ships, players still launch `client-dev` and are **unreleasable**.

Settings live under **Edit → Project Settings → Tryll Client**. **Auto Launch Server** is `true` by default — uncheck it to manage the server yourself. **Llama.cpp backend** (`Vulkan` / `Auto`, default Vulkan) is passed as `--llama-backend` and also decides whether a player build copies the CUDA DLLs.

Set `TRYLL_SERVER_MODE=tryll-dev` to launch Tryll Dev with `--disable-telemetry` (Tryll’s own CI / developers).

`TryllClient` spawns the server before attempting to connect and terminates the process when the application quits.

Bundling the server

The server binaries ship inside the package and are launched automatically — in the Editor from the package, and in a player build from the `Server/` folder copied next to the player exe. Production models staged from `TryllStorage/` travel with the build.

______________________________________________________________________

## Unreal

The Unreal plugin ships the server **inside the plugin** and auto-launches it — there is no path to configure.

1. Open **Edit → Project Settings → Plugins → Tryll Client**.
1. **Auto Launch Server** is `true` by default. Uncheck it to disable auto-launch entirely and run a server yourself (e.g. a local debug build).
1. **Llama.cpp backend** (`Vulkan` / `Auto`, default Vulkan) is passed as `--llama-backend`. `Auto` prefers CUDA on NVIDIA and ships the CUDA DLLs in a packaged build; `Vulkan` skips them.
1. **Server Build Variant** (only shown when both a `Default` and a `Release` variant resolve — a plugin-development setup) picks which build to launch. With a single variant present it is used automatically and this option is hidden.

In the editor and in **Play → Standalone Game** the subsystem runs `<Plugin>/Binaries/ThirdParty/TryllServer/<variant>/tryll_server.exe`. Packaging copies that variant next to the game, and a packaged game runs `<Project>/Server/tryll_server.exe`. The subsystem spawns the server on `Initialize` with `--mode client-dev`, `--project-name`, `--project-dir`, and `--storage-root` (default `<Project>/TryllStorage/`, outside `Content/`). On `Deinitialize` it lets go of the process, and the server exits by itself 60 s after its last session ends. **Llama.cpp backend** is passed as `--llama-backend`, and the port as `--port`, so both are **authoritative** and override `server-config.json` — you do **not** need to keep the two in sync.

Who owns the server and which port it uses depends on how the game runs:

| Run                          | Server                                                | Port                                                               |
| ---------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------ |
| Play in Editor, editor tools | One server shared by the editor and every PIE session | **ServerPort** (default 9100), or a free port if that one is taken |
| Play → Standalone Game       | Its own server, separate from the editor's            | A free port                                                        |
| Packaged game                | Its own server                                        | A free port                                                        |

Set `TRYLL_SERVER_MODE=tryll-dev` to launch Tryll Dev with `--disable-telemetry`. Until Client Prod ships, packaged players still launch `client-dev` and are **unreleasable**.

Where the server comes from

The plugin's `Binaries/ThirdParty/TryllServer/` holds committed `Default` / `Release` symlinks to your local server builds for editor iteration (they dangle until you build the server, then resolve); the release pipeline replaces `Default` with a real production copy so the server ships with packaged builds. You never point the plugin at an external exe. `TryllClient.Build.cs` stages `TryllStorage/` as NonUFS.

______________________________________________________________________

## C++

### Recommended: `TryllClient::RunAndConnect`

`RunAndConnect` is the one-call factory for the common case. It starts the server, waits for the TCP port to open, and returns a `ConnectedSession` that owns both the process and the socket — destructor shuts them down in the right order automatically.

```
#include <tryll/TryllClient.h>   // pulls in ManagedServer.h transitively

namespace TC = Tryll::Client;

TC::ManagedServerOptions opts;
opts.exe  = "C:/tryll/tryll_server.exe";  // required — no auto-discovery
opts.port = 9100;

// Spawn server → wait for TCP → connect → return session.  Throws on failure.
auto session = Tryll::Client::TryllClient::RunAndConnect(opts);

session.GetClient().CreateSession({});
auto agent = session.GetClient().CreateAgent(graph);
// …
// session destructor: closes the client connection. The now-idle server is
// NOT hard-killed — it self-exits after opts.idleShutdownTimeoutSeconds
// (default 60 s) once it has zero sessions and no in-flight downloads.
```

For async startup (e.g. editor plugin startup, game initialization):

```
auto future = Tryll::Client::TryllClient::RunAndConnectAsync(std::move(opts));
// do other work …
auto session = future.get();   // blocks until ready; propagates any TryllError
```

### Lower-level: `ManagedServer` + `Connect`

When you need finer control over the server lifetime (e.g. a long-lived server shared across multiple client sessions), use `ManagedServer` and `Connect` separately:

```
#include <tryll/ManagedServer.h>
#include <tryll/TryllClient.h>

TC::ManagedServerOptions opts;
opts.exe  = "C:/tryll/tryll_server.exe";
opts.port = 9100;

auto server = TC::ManagedServer::Start(opts);
// Start() blocks until the TCP port accepts connections (up to opts.startTimeout).

// First session
auto clientA = Tryll::Client::TryllClient::Connect(server.Host(), server.Port());
clientA.CreateSession({});
// … use clientA …
clientA.Shutdown();

// Second session — server still running
auto clientB = Tryll::Client::TryllClient::Connect(server.Host(), server.Port());
// …
clientB.Shutdown();

server.Stop();   // or let server go out of scope — destructor calls Stop()
```

Stop() does not hard-kill

`ManagedServer::Stop()` (and the destructor) close the client's side and let the server **self-exit** once it is idle for `idleShutdownTimeoutSeconds` (default 60 s) — it does not forcibly terminate the process. Set `idleShutdownTimeoutSeconds = 0` only if you own an explicit reap strategy (e.g. a test harness that hard-kills on teardown).

### Key `ManagedServerOptions` fields

| Option                       | Default             | Description                                                                                                                                                                   |
| ---------------------------- | ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `exe`                        | *(required)*        | Path to `tryll_server.exe`. No automatic discovery.                                                                                                                           |
| `port`                       | `9100`              | Passed as `--port` to the server (overrides `server-config.json`).                                                                                                            |
| `idleShutdownTimeoutSeconds` | `60`                | Passed as `--idle-shutdown-timeout` when non-zero: the launched server self-exits after this many seconds idle. `0` = never self-exit (use only with your own reap strategy). |
| `extraArgs`                  | *(none)*            | Extra command-line arguments appended after `--port <port>`.                                                                                                                  |
| `host`                       | `"127.0.0.1"`       | Used only for the TCP ready-probe.                                                                                                                                            |
| `workingDirectory`           | `exe.parent_path()` | Where the server looks for `data/`.                                                                                                                                           |
| `stdoutLog` / `stderrLog`    | *(discard)*         | Redirect server output to files.                                                                                                                                              |
| `startTimeout`               | 30 s                | How long to wait for the port to open.                                                                                                                                        |
| `stopTimeout`                | 8 s                 | How long to wait for graceful exit on `Stop()`.                                                                                                                               |

### test-chat

The bundled `tryll_test_chat` demo uses `RunAndConnect` automatically and discovers the server exe from the build directory. Use `--no-managed-server` to disable this and connect to a server you started yourself:

```
test-chat\build\Debug\tryll_test_chat.exe --no-managed-server
```

Other useful flags:

```
# Use a server exe from a different location
tryll_test_chat.exe --server-exe C:\tryll\tryll_server.exe

# Run on a non-default port
tryll_test_chat.exe --server-port 9200
```

______________________________________________________________________

## Python

### Recommended: `TryllClient.run_and_connect`

`run_and_connect` is the one-call factory for the common case. It starts the server and returns a `ConnectedSession` context manager that tears everything down automatically:

```
from pathlib import Path
from tryll_client import TryllClient

with TryllClient.run_and_connect(
    exe=Path("C:/tryll/tryll_server.exe"),  # required — no auto-discovery
    port=9100,
) as session:
    session.client.create_session()
    agent = session.client.create_agent(graph)
    reply = agent.send_message("Hello!")
    print(reply)
# __exit__: client.shutdown() then server.stop()
```

Without a context manager:

```
session = TryllClient.run_and_connect(exe=Path("C:/tryll/tryll_server.exe"), port=9100)
try:
    session.client.create_session()
    # …
finally:
    session.shutdown()
```

### Lower-level: `ManagedServer` + `connect`

When you need a long-lived server shared across multiple client sessions, use `ManagedServer` and `connect` separately:

```
from pathlib import Path
from tryll_client import TryllClient, ManagedServer

with ManagedServer.start(
    exe=Path("C:/tryll/tryll_server.exe"),  # required — no auto-discovery
    port=9100,
) as srv:
    # First session
    client_a = TryllClient.connect(srv.host, srv.port)
    client_a.create_session()
    # … use client_a …
    client_a.shutdown()

    # Second session — server still running
    client_b = TryllClient.connect(srv.host, srv.port)
    # …
    client_b.shutdown()
# ManagedServer.__exit__ calls stop() automatically
```

### Key parameters

| Parameter               | Default       | Description                                                                                                                                                    |
| ----------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `exe`                   | *(required)*  | Path to `tryll_server[.exe]`.                                                                                                                                  |
| `port`                  | `9100`        | Passed as `--port` to the server (overrides `server-config.json`).                                                                                             |
| `idle_shutdown_timeout` | `60`          | Passed as `--idle-shutdown-timeout` when non-zero: the launched server self-exits after this many seconds idle. `0` = never (use with your own reap strategy). |
| `extra_args`            | *(none)*      | Extra command-line arguments appended after `--port <port>`.                                                                                                   |
| `host`                  | `"127.0.0.1"` | Used only for the TCP ready-probe.                                                                                                                             |
| `cwd`                   | `exe.parent`  | Working directory for the child process.                                                                                                                       |
| `stdout` / `stderr`     | *(discard)*   | Redirect server output to files.                                                                                                                               |
| `start_timeout`         | `30.0`        | Seconds to wait for the port to open.                                                                                                                          |
| `stop_timeout`          | `8.0`         | Seconds to wait for graceful exit on `stop()`.                                                                                                                 |
| `connect_timeout`       | `30.0`        | (`run_and_connect` only) Seconds to wait for `ConnectionReady`.                                                                                                |

______________________________________________________________________

## Related

- [Run the Tryll Server](https://docs.tryllengine.com/v0.9/how-to/run-the-tryll-server.html) — manual server launch and `server-config.json` reference.
- [C++ Client API](https://docs.tryllengine.com/v0.9/reference/client-api/cpp/index.html) — full `ManagedServer` class reference.
- [Python Client API](https://docs.tryllengine.com/v0.9/reference/client-api/python/index.html) — full `ManagedServer` class reference.
- [Reference: Server Configuration](https://docs.tryllengine.com/v0.9/reference/server-configuration.html) — `--port` and `--config` CLI flags.
