# Connect and Manage a Session

Open a connection to a running Tryll server, handle reconnection, and shut down cleanly.

Prerequisites

- The Tryll server is running and reachable — see [Run the Tryll Server](https://docs.tryllengine.com/how-to/run-the-tryll-server.html).
- You know the host and port (default `127.0.0.1:9100`).

## Steps

```
// 1. Subscribe to connection events before connecting.
TryllClient.Instance.ConnectionChanged += connected => {
    if (!connected) return;
    // 2. Configure the inference engine for this session.
    TryllClient.Instance.CreateSession(TryllInferenceEngine.LlamaCpp);
};
TryllClient.Instance.CreateSessionComplete += err => {
    if (!err.IsOk)
        Debug.LogError($"configure failed: {err.Message}");
    // 3. Proceed to create storages and agents.
};

// TryllClient connects automatically on Awake when Auto Launch Server
// is configured in Project Settings. To connect manually:
TryllClient.Instance.Connect();
```

In Blueprint: on `Event BeginPlay`, get the `UTryllSubsystem` and call **Connect**. Bind **On Connection Changed** — when `bConnected = true`, call **Create Session** with your chosen `ETryllInferenceEngine` and wait for **On Create Session Complete**.

In C++, the same subsystem API is available:

```
auto* subsystem = GetGameInstance()->GetSubsystem<UTryllSubsystem>();
subsystem->OnConnectionChanged.AddDynamic(this, &ThisClass::HandleConnection);
subsystem->Connect();
```

```
#include <tryll/TryllClient.h>

namespace TC = Tryll::Client;

try
{
    auto client = Tryll::Client::TryllClient::Connect("127.0.0.1", 9100);
    client.CreateSession({ .engine = ::Tryll::InferenceEngine_LlamaCpp });

    // ... create agents, send messages ...

    client.Shutdown();
}
catch (const Tryll::Client::TryllError& ex)
{
    // ex.GetCode() is an ErrorCode; see reference/error-codes.md
    std::cerr << "session error " << ex.GetCode()
              << ": " << ex.what() << "\n";
}
```

```
from tryll_client import TryllClient, TryllError, InferenceEngine

# 1. Open the socket. Blocks briefly until ConnectionReady arrives.
client = TryllClient.connect("127.0.0.1", 9100)

# 2. Pick the inference engine for this session.
#    Affects agents you create *after* this call.
try:
    client.create_session(InferenceEngine.LlamaCpp)
except TryllError as e:
    print(f"configure failed: {e}")
    raise

# 3. Do your work — create storages, agents, send messages.
#    See build-chat-agent-with-graph.md for the next step.

# 4. Clean shutdown.
client.shutdown()
```

## Handle reconnection

If the socket drops (server restart, network blip, etc.), the server destroys every agent and storage the session owned. The client libraries surface this as a connection-changed event or an async error:

- **Unity / Unreal:** bind `ConnectionChanged` / `OnConnectionChanged(false)` to a handler that re-calls `Connect` / `CreateSession` and re-creates storages and agents.
- **Python / C++:** catch the `TryllError` from the next request and call `TryllClient.connect(...)` / `Tryll::Client::TryllClient::Connect(...)` again to make a fresh client. The old client's storages and agents are gone; re-create them.

There is no server-side session persistence. Everything is rebuilt from scratch on reconnect.

```
// ConnectionChanged fires false on drop, true when re-established.
TryllClient.Instance.ConnectionChanged += connected => {
    if (!connected) return;
    TryllClient.Instance.CreateSession(TryllInferenceEngine.LlamaCpp);
    // Re-create any per-session storages and agents you need.
};
```

```
// Bind OnConnectionChanged(false) to a handler that re-calls
// Connect after a short delay, then re-runs your CreateSession
// + graph/storage bootstrap when bConnected flips back to true.
auto* subsystem = GetGameInstance()->GetSubsystem<UTryllSubsystem>();
subsystem->Connect();
```

```
Tryll::Client::TryllClient reconnect()
{
    auto client = Tryll::Client::TryllClient::Connect("127.0.0.1", 9100);
    client.CreateSession({ .engine = ::Tryll::InferenceEngine_LlamaCpp });
    // re-create any per-session storages and agents you need
    return client;
}
```

```
def reconnect() -> TryllClient:
    client = TryllClient.connect("127.0.0.1", 9100)
    client.create_session(InferenceEngine.LlamaCpp)
    # re-create any per-session storages and agents you need
    return client
```

## Session lifetime rules to remember

- `CreateSession` is **mandatory and one-shot** (protocol v4): call it exactly once after connecting, before any agent/model/storage/voice request. The chosen engines are fixed for the session's lifetime. A request before `CreateSession` fails with `SessionNotReady`; a second `CreateSession` fails with `SessionAlreadyExists`. To change engines, reconnect and create a new session.
- String storages and embedded string storages die with the session.
- Model downloads and loads are *global*, not per-session. Pinning a model in one session keeps it warm for all sessions.
- Agent ids are session-scoped. A stored `agent_id` is meaningless after reconnect.

See [Lifetime and Ownership](https://docs.tryllengine.com/concepts/lifetime-and-ownership.html) for the full picture of what survives session close and what does not.

## Suppress telemetry for a process run

To prevent a server from emitting telemetry events (e.g. to PostHog) for a specific run — such as a QA or CI pipeline — pass `--disable-telemetry` on the server command line:

```
tryll_server.exe --disable-telemetry
```

This overrides `"enabled": true` in `server-config.json` for that process lifetime. See [Telemetry — CLI override](https://docs.tryllengine.com/reference/telemetry.html#suppress-telemetry-for-a-process-run-cli-override) for details.

## Verify it worked

Connecting and configuring produces these server-side log lines (at `info` level):

```
[info] Session 42 accepted 127.0.0.1:56102
[info] [Session 42] Configured engines: language=LlamaCpp  stt=Mock  tts=Mock  embedding=Mock  game_name:
```

If you do not see a configure line, check that `OnCreateSessionComplete` fired with `bSuccess = true` on the client. A failure is almost always a server-side inference-engine issue — see [error codes 2xxx](https://docs.tryllengine.com/reference/error-codes.html#2xxx-session).

## Related

- [Reference: Wire Protocol](https://docs.tryllengine.com/reference/wire-protocol.html)
- [Reference: Agent Parameters](https://docs.tryllengine.com/reference/agent-parameters.html)
- [Reference: Error Codes](https://docs.tryllengine.com/reference/error-codes.html)
- [Concept: Agents and Sessions](https://docs.tryllengine.com/concepts/agents-and-sessions.html)
- [Concept: Lifetime and Ownership](https://docs.tryllengine.com/concepts/lifetime-and-ownership.html)
- [How-to: Build a Chat Agent with a Graph](https://docs.tryllengine.com/how-to/build-chat-agent-with-graph.html)
