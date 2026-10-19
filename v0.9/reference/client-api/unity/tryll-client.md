# TryllClient

**Type:** `MonoBehaviour` (singleton)\
**Namespace:** `Tryll.Client`\
**Source:** `Runtime/TryllClient.cs`

Singleton `MonoBehaviour` that manages the TCP session to the Tryll server. Created automatically at game start by `TryllClientModule` via `[RuntimeInitializeOnLoadMethod(BeforeSceneLoad)]` — you do not add it to a scene yourself.

In Play mode it lives on a `DontDestroyOnLoad` GameObject. In the Editor outside Play mode the static helper `EnsureEditorInstance()` creates a hidden `HideAndDontSave` instance.

______________________________________________________________________

## Connection settings

These are read from [`TryllRuntimeSettings`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-runtime-settings.html) on `Awake`.

| Field        | Type     | Default       |
| ------------ | -------- | ------------- |
| `ServerHost` | `string` | `"127.0.0.1"` |
| `ServerPort` | `int`    | `9100`        |

______________________________________________________________________

## Public API

```
// Singleton access
public static TryllClient Instance { get; }

// Connection state
public bool  IsConnected { get; }
public ulong SessionId   { get; }

// Connection
public void Connect();
public void Disconnect();

// Session
public void CreateSession((byte Role, byte Engine)[] engines,   // optional warm-up hint
                          string gameName = "",
                          string locale = null,             // null = project setting
                          (string Model, byte Placement)[] placementHints = null,
                          (string Model, byte Hotwords)[] hotwordHints = null);
// A model's engine comes from its catalog entry. Settings.BuildEngineDeclarations()
// builds the hint from the Model Manager manifest; an empty array is legal.
// Storage root is process-wide (Project Settings → Storage Data Folder → --storage-root).
// Protocol v24 removed CreateSessionRequest.storage_data_folder.

// Agent lifecycle — always check error.IsOk before using the agent
public Task<(TryllAgent agent, TryllError error)>
    RequestCreateAgentAsync(
        TryllGraphDescription graph,
        bool enableDiagnostics = false,
        bool maintainDialogueHistory = true,
        List<TryllVariableDecl> variables = null,
        Tryll.AgentKvCacheInitialization kvCacheInitialization = Tryll.AgentKvCacheInitialization.AllocateOnly,
        Tryll.AgentWorkload workload = Tryll.AgentWorkload.Interactive,
        string textTable = null,
        string assetName = null,
        TryllVoiceProfileData voice = null);   // required when a node speaks

// Model management
public Task<(List<TryllModelInfo> models, TryllError error)> RequestListModelsAsync();
public Task<(TryllMemorySnapshot snapshot, TryllError error)> RequestGetMemoryConsumptionAsync();
public void RequestDownloadModel(string modelName);
public void RequestLoadModel(string modelName, TryllMemoryPlacement? placement = null);
public void RequestUnloadModel(string modelName);

// StringStorage (List / Map / Multimap)
public Task<TryllError> RequestCreateStringStorageAsync(string name, List<string> strings);
public Task<TryllError> RequestCreateStringStorageFromFileAsync(
    string name, string filePath, Tryll.StringStorageKind kind = Tryll.StringStorageKind.List);
public Task<TryllError> RequestCreateKeyedStringStorageAsync(
    string name, List<string> keys, List<string> values,
    Tryll.StringStorageKind kind = Tryll.StringStorageKind.Map);
public Task<TryllError> RequestDestroyStringStorageAsync(string name);

// EmbeddedStringStorage (RAG)
public Task<(EmbeddedStorageInfo info, TryllError error)>
    RequestCreateEmbeddedStringStorageAsync(string name, string configPath, string embeddingModel = "");
public Task<(EmbeddedStorageInfo info, TryllError error)>
    RequestCreateEmbeddedStringStorageFromStringsAsync(string name, List<string> strings, string embeddingModel);
public Task<TryllError> RequestDestroyEmbeddedStringStorageAsync(string name);

// Voice input (STT) — see the TryllVoiceInput handle for BeginUtterance/EndUtterance/…
public Task<(TryllVoiceInput voice, TryllError error)> CreateVoiceInputAsync(VoiceInputConfig cfg);

// Experimental mutable knowledge bases
public Task<(TryllMutableKb store, TryllError error)> EnsureMutableKbAsync(string name, string schema);
public Task<(List<TryllMutableKbInfo> stores, TryllError error)> EnumerateMutableKbsAsync();
public Task<(TryllGameMemorySaveResult result, TryllError error)> SaveGameMemoryAsync(string path, string saveId);
public Task<(TryllGameMemoryLoadResult result, TryllError error)> LoadGameMemoryAsync(
    string path, TryllMutableKbEmbedTiming embedTiming,
    TryllMutableKbCompatibility compatibility = TryllMutableKbCompatibility.Strict);

// GPU throttling — fire-and-forget, no response, safe to call every frame
public void SetInferenceThrottle(float level);

// Editor utility
public static TryllClient EnsureEditorInstance();
```

`RequestGetMemoryConsumptionAsync` is session-scoped (needs `CreateSession`) but the snapshot is **server-wide** — every session on that process. It never fails with `AgentBusy`. See [Inspect Server Memory](https://docs.tryllengine.com/v0.9/how-to/inspect-server-memory.html).

______________________________________________________________________

## GPU throttling and background agents

`SetInferenceThrottle` tells the server how hard to yield the GPU back to your game — `0f` = full speed (the default for a session that never calls it), `1f` = maximum yielding. It is fire-and-forget: no request id, no response, nothing to await, so calling it from `Update()` is fine. It never changes what is generated, only how fast.

```
void Update()
{
    // Example policy: yield harder as your own frame time worsens.
    float pressure = Mathf.InverseLerp(11f, 20f, Time.smoothDeltaTime * 1000f);
    TryllClient.Instance.SetInferenceThrottle(pressure);
}
```

Agents nobody is waiting on should say so, so their inference is the first to give way:

```
var (agent, error) = await TryllClient.Instance.RequestCreateAgentAsync(
    graph, workload: Tryll.AgentWorkload.Background);
```

Everything else the server classifies from the graph itself, per turn — streamed `Generate` text is protected, spoken and buffered work absorbs the slack. See [Wire Protocol](https://docs.tryllengine.com/v0.9/reference/wire-protocol.html#inference-throttle-and-agent-workload) and [Server configuration](https://docs.tryllengine.com/v0.9/reference/server-configuration.html#inference-scheduler).

______________________________________________________________________

## Events

All events fire on the Unity **main thread**.

| Event                          | Signature                                                                   | Description                                                                                                                                                                                                                                                         |
| ------------------------------ | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ConnectionChanged`            | `Action<bool>`                                                              | Connection state changed. Parameter: `isConnected`.                                                                                                                                                                                                                 |
| `Error`                        | `Action<TryllError>`                                                        | Session-level or unroutable error.                                                                                                                                                                                                                                  |
| `CreateSessionComplete`        | `Action<TryllError>`                                                        | Server acknowledged `CreateSession`.                                                                                                                                                                                                                                |
| `DownloadProgress`             | `Action<string, ulong, ulong, float>`                                       | `(modelName, bytesDownloaded, totalBytes, percent 0–100)` — streaming progress. `totalBytes` may be `0` when unknown.                                                                                                                                               |
| `DownloadComplete`             | `Action<string, bool, string>`                                              | `(modelName, success, error)` — download finished or failed. `error` is empty on success.                                                                                                                                                                           |
| `LoadModelComplete`            | `Action<string, bool>`                                                      | `(modelName, success)`                                                                                                                                                                                                                                              |
| `UnloadModelComplete`          | `Action<string, bool>`                                                      | `(modelName, success)`                                                                                                                                                                                                                                              |
| `AgentDestroyed`               | `Action<ulong>`                                                             | `(agentId)` — server confirmed agent teardown.                                                                                                                                                                                                                      |
| `ToolCallNotification`         | `Action<ulong, string, string>`                                             | All agents. `(agentId, toolName, argumentsJson)`. Prefer `TryllAgentComponent.ToolCallNotification` for one NPC.                                                                                                                                                    |
| `ToolCallNotificationWithId`   | `Action<ulong, string, string, string>`                                     | All agents. `(agentId, callId, toolName, argumentsJson)`.                                                                                                                                                                                                           |
| `ToolCallDetailed`             | `Action<ulong, TryllToolCall>`                                              | All agents. Fired in addition to the other tool-call events; a subscriber does not swallow `NodeEvent`.                                                                                                                                                             |
| `IntentClassified`             | `Action<ulong, TryllIntentClassification>`                                  | All agents. `(agentId, classification)`. The payload has no agent id. A subscriber swallows this client's generic `NodeEvent`.                                                                                                                                      |
| `IntentLlmClassified`          | `Action<ulong, TryllIntentLlmClassification>`                               | All agents. `(agentId, classification)`. The payload has no agent id. A subscriber swallows this client's generic `NodeEvent`.                                                                                                                                      |
| `Paused`                       | `Action<ulong, string, string>`                                             | All agents. `(agentId, nodeName, pendingExit)` — the executor paused a turn between nodes (`Pause` node, or a `ToolCall` with a pausing `disposition`). See [How to pause and resume a turn](https://docs.tryllengine.com/v0.9/how-to/pause-and-resume-turns.html). |
| `NodeEvent`                    | `Action<ulong, string, string, IReadOnlyList<KeyValuePair<string,string>>>` | All-agents generic fallback for unknown or unsubscribed typed events. `(agentId, nodeName, eventType, kvPairs)`. An agent-scope listener does not suppress this, and the reverse.                                                                                   |
| `StringStorageChanged`         | `Action<string, bool>`                                                      | `(name, isCreate)`                                                                                                                                                                                                                                                  |
| `EmbeddedStringStorageChanged` | `Action<string, int, bool>`                                                 | `(name, recordCount, isCreate)`                                                                                                                                                                                                                                     |

______________________________________________________________________

## EmbeddedStorageInfo

```
public struct EmbeddedStorageInfo
{
    public string Name;
    public uint   RecordCount;
    public uint   EmbeddingDim;
}
```

______________________________________________________________________

## Lifecycle

1. `TryllClientModule` creates the singleton at `BeforeSceneLoad`.
1. `Awake` reads `TryllRuntimeSettings` and applies `ServerHost` / `ServerPort`.
1. Call `Connect()` to open the TCP session (or enable **Auto Launch Server** in settings).
1. Call `CreateSession(...)` to open the session. It may pass an optional engine warm-up hint; it does not choose an engine. Each model's engine comes from its catalog entry.
1. Use `RequestCreateAgentAsync` or attach a [`TryllAgentComponent`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-agent-component.html).
1. Call `Disconnect()` when done, or let `OnDestroy` clean up on exit.

______________________________________________________________________

## See also

- [`TryllAgentComponent`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-agent-component.html) — drop-in GameObject component
- [`TryllRuntimeSettings`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-runtime-settings.html) — server path and host/port
- [Connect and manage a session](https://docs.tryllengine.com/v0.9/how-to/connect-and-manage-session.html)
- [Model Management](https://docs.tryllengine.com/v0.9/reference/model-management.html)
