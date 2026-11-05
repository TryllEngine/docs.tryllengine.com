# Connect and Manage a Session

Open a connection to a running Tryll server, handle reconnection, and shut down cleanly.

Prerequisites

- The Tryll server is running and reachable — see [Run the Tryll Server](https://docs.tryllengine.com/v0.10/how-to/run-the-tryll-server.html).
- You know the host and port. A server you start by hand binds the `server-config.json` port (`9100` by default), which the C++ and Python samples below assume. Auto-launched player servers, and a `ManagedServer` left at its default port, use an OS-assigned free port instead: connect through the client that launched them, not to 9100. In the editor, 9100 is the configured port and the fallback the plugin tries first.

## Steps

```
// 1. Subscribe to connection events before connecting.
TryllClient.Instance.ConnectionChanged += connected => {
    if (!connected) return;
    // 2. Create the session. The engine hint comes from the Model
    //    Manager manifest; each model's catalog entry names its engine.
    TryllClient.Instance.CreateSession(
        TryllClient.Instance.Settings.BuildEngineDeclarations());
};
TryllClient.Instance.CreateSessionComplete += err => {
    if (!err.IsOk)
        Debug.LogError($"configure failed: {err.Message}");
    // 3. Proceed to create storages and agents.
};

// Auto Launch Server (Project Settings) only starts the server process.
// Auto Start Session on a scene TryllClient (off by default) connects and
// creates the session from Awake; StartSession() does the same on demand.
// Otherwise connect manually:
TryllClient.Instance.Connect();
```

In Blueprint: on `Event BeginPlay`, get the `UTryllSubsystem` and call **Connect**. Bind **On Connection Changed** — when `bConnected = true`, call **Create Session** and wait for **On Create Session Complete**. (With **Auto Create Session** on, the default, the plugin does this for you.)

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
    client.CreateSession({});

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
from tryll_client import TryllClient, TryllError

# 1. Open the socket. Blocks briefly until ConnectionReady arrives.
client = TryllClient.connect("127.0.0.1", 9100)

# 2. Create the session (mandatory, one-shot). It does not pick an
#    engine: each model's catalog entry names its engine. The optional
#    `engines` hint only warms engines early.
try:
    client.create_session()
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
    TryllClient.Instance.CreateSession(
        TryllClient.Instance.Settings.BuildEngineDeclarations());
    // Re-create any per-session storages and agents you need.
};
```

A dropped connection ends the session. Call **Connect** again, then make a new session. With **Auto Create Session** (`bAutoCreateSession`, on by default) the subsystem creates it when the connection comes back, and every `UTryllAgentComponent` with **Auto Create On Connect** re-creates its agent once the session is ready. Storages and agents you created from code must be re-created in `OnCreateSessionComplete`.

```
// Bound with: Subsystem->OnConnectionChanged.AddDynamic(this, &ThisClass::HandleConnection);
//             Subsystem->OnCreateSessionComplete.AddDynamic(this, &ThisClass::HandleSession);
void AMyActor::HandleConnection(bool bConnected)
{
    UTryllSubsystem* Subsystem = GetGameInstance()->GetSubsystem<UTryllSubsystem>();
    if (!bConnected)
    {
        // Try again after a short delay.
        GetWorldTimerManager().SetTimer(ReconnectTimer,
            FTimerDelegate::CreateWeakLambda(this, [Subsystem] { Subsystem->Connect(); }),
            2.0f, false);
        return;
    }
    if (!GetDefault<UTryllRuntimeSettings>()->bAutoCreateSession)
    {
        Subsystem->CreateSessionFromSettings();
    }
}

void AMyActor::HandleSession(bool bSuccess)
{
    if (bSuccess)
    {
        // Re-create string storages, embedded storages, and code-created agents here.
    }
}
```

```
Tryll::Client::TryllClient reconnect()
{
    auto client = Tryll::Client::TryllClient::Connect("127.0.0.1", 9100);
    client.CreateSession({});
    // re-create any per-session storages and agents you need
    return client;
}
```

```
def reconnect() -> TryllClient:
    client = TryllClient.connect("127.0.0.1", 9100)
    client.create_session()
    # re-create any per-session storages and agents you need
    return client
```

## Session lifetime rules to remember

- `CreateSession` is **mandatory and one-shot** (protocol v4): call it exactly once after connecting, before any agent/model/storage/voice request. A request before `CreateSession` fails with `SessionNotReady`; a second `CreateSession` fails with `SessionAlreadyExists`. There is no engine to change: each model's catalog entry names its engine, so a model you download mid-session loads without a new session.
- String storages and embedded string storages die with the session.
- Model downloads and loads are *global*, not per-session. Pinning a model in one session keeps it warm for all sessions.
- Agent ids are session-scoped. A stored `agent_id` is meaningless after reconnect.

See [Lifetime and Ownership](https://docs.tryllengine.com/v0.10/concepts/lifetime-and-ownership.html) for the full picture of what survives session close and what does not.

## Verify it worked

Connecting and creating the session produces these server-side log lines (at `info` level):

```
[info] [Connection 3] accepted
[info] [Session 1] Created  uuid=<session-uuid>  engine hint: (none)  game_name: MyGame  storage_root: C:\MyGame\TryllStorage  locale: (unset)
```

The `engine hint` field lists `role=engine` pairs (for example `language=llama-cpp, tts=sherpa-onnx`) when the client sent a warm-up hint, and `(none)` when it did not. `storage_root` is the process storage root.

If you do not see a `Created` line, `CreateSession` did not complete. Check the error the client received (`CreateSessionComplete` in Unity, **On Create Session Complete** in Unreal, the raised `TryllError` in C++ and Python) — see [error codes 2xxx](https://docs.tryllengine.com/v0.10/reference/error-codes.html#2xxx-session).

## Related

- [Reference: Wire Protocol](https://docs.tryllengine.com/v0.10/reference/wire-protocol.html)
- [Reference: Agent Parameters](https://docs.tryllengine.com/v0.10/reference/agent-parameters.html)
- [Reference: Error Codes](https://docs.tryllengine.com/v0.10/reference/error-codes.html)
- [Concept: Agents and Sessions](https://docs.tryllengine.com/v0.10/concepts/agents-and-sessions.html)
- [Concept: Lifetime and Ownership](https://docs.tryllengine.com/v0.10/concepts/lifetime-and-ownership.html)
- [How-to: Build a Chat Agent with a Graph](https://docs.tryllengine.com/v0.10/how-to/build-chat-agent-with-graph.html)
