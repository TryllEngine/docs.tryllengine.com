# TryllRuntimeSettings

**Type:** `ScriptableObject`\
**Namespace:** `Tryll.Client`\
**Source:** `Runtime/TryllRuntimeSettings.cs`

Project-level settings for the Tryll Client package. Persisted to `Assets/Resources/TryllRuntimeSettings.asset` (created automatically on first access).

Edit via **Project Settings → Tryll Client** (registered by `TryllRuntimeSettingsProvider`).

______________________________________________________________________

## Loading at runtime

```
var settings = TryllRuntimeSettings.Load();
```

`Load()` returns a cached singleton. If the asset does not exist in `Resources`, it falls back to default values.

______________________________________________________________________

## Fields

### Server process

| Field                | Type          | Default   | Description                                                                                                                                                 |
| -------------------- | ------------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `AutoLaunchServer`   | `bool`        | `true`    | When true, `TryllClient` spawns the bundled `tryll_server.exe` automatically on startup (Windows only). Set to `false` when you manage the server yourself. |
| `ServerBuildVariant` | `ServerBuild` | `Default` | Which bundled server payload to launch/stage: `Default` or `Release`.                                                                                       |

The server exe is always resolved from the package payload — bundled in the package in the Editor, or the `Server/` folder copied next to the player exe at build time — so there is **no path to configure**.

### Connection defaults

| Field        | Type     | Default       | Description                                  |
| ------------ | -------- | ------------- | -------------------------------------------- |
| `ServerHost` | `string` | `"127.0.0.1"` | Host name or IP address of the Tryll server. |
| `ServerPort` | `int`    | `9100`        | TCP port the Tryll server listens on.        |

### Session engine defaults

Used by `TryllClient.StartSession()` / `CreateSessionFromSettings()` when you don't pass engines explicitly.

| Field             | Type                   | Default    | Description                                                |
| ----------------- | ---------------------- | ---------- | ---------------------------------------------------------- |
| `InferenceEngine` | `TryllInferenceEngine` | `LlamaCpp` | Language-model backend.                                    |
| `SttEngine`       | `TryllInferenceEngine` | `Mock`     | Speech-to-text backend. Set `SherpaOnnx` for voice input.  |
| `TtsEngine`       | `TryllInferenceEngine` | `Mock`     | Text-to-speech backend. Set `SherpaOnnx` for voice output. |
| `EmbeddingEngine` | `TryllInferenceEngine` | `Mock`     | Embedding backend for RAG / intent.                        |

### Storage

| Field               | Type     | Default | Description                                                                                                                                                                                     |
| ------------------- | -------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `StorageDataFolder` | `string` | `""`    | Project-relative root that relative node storage paths and voice-input hotword files resolve against (sent as `storage_data_folder` on `CreateSession`). Empty = the server's own storage root. |

### Editor / Play Mode

| Field                      | Type   | Default | Description                                                                                                                                         |
| -------------------------- | ------ | ------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `EnablePlayModeModelGuard` | `bool` | `true`  | When true, entering Play Mode (or the Chat window) verifies every registered model is downloaded first, blocking behind a cancelable dialog if not. |

### Telemetry

| Field      | Type     | Default | Description                                                                                                                                       |
| ---------- | -------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `GameName` | `string` | `""`    | Integration identifier sent on every session for telemetry grouping. Use a short, stable, lowercase slug (e.g. `"my-game"`). Leave empty to omit. |

______________________________________________________________________

## See also

- [`TryllClient`](https://docs.tryllengine.com/reference/client-api/unity/tryll-client.html) — reads these settings on `Awake`
- [Run the Tryll server](https://docs.tryllengine.com/how-to/run-the-tryll-server.html)
- [Manage models in the editor](https://docs.tryllengine.com/how-to/manage-models-in-editor.html)
- [Telemetry](https://docs.tryllengine.com/reference/telemetry.html)
