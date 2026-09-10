# Ship Storage Folders for Builds

Tryll nodes that use relative `string_storage` or `embedded_string_storage` paths need the storage files to be readable by the **server process** at runtime. This guide explains the file layout conventions, how to ship those files with each client platform, and how to tell the server where to find them via `storage_data_folder`.

## File layout conventions

Organize your storage files under a single root folder that you will ship alongside (or pointing at) the server:

```
your_game/
  StorageData/
    responses/
      refusal_lines.txt          # CannedResponse — List kind
      polite_reject.txt
    intent/
      npc_instructions.json      # IntentToInstruction — Map kind
    rag/
      adventure_quests.json      # EmbeddedStringStorage config (Path A)
      adventure_quests.kb.json   # records file (referenced from config)
      adventure_quests.usearch   # optional pre-built HNSW index
    hotwords/
      wake_words.txt             # voice hotwords — List kind
```

Node params reference files **relative to the storage root**:

```
string_storage          = "responses/refusal_lines.txt"
embedded_string_storage = "rag/adventure_quests.json"
hotwords_storage_path   = "hotwords/wake_words.txt"
```

### File formats

| Extension             | Kind                          | Format                                                                                |
| --------------------- | ----------------------------- | ------------------------------------------------------------------------------------- |
| `.txt`                | List                          | UTF-8 text, one value per line. Lines starting with `#` and blank lines are skipped.  |
| `.json`               | Map                           | JSON array of `{"id": "key", "text": "value"}` objects.                               |
| `.mm.json`            | Multimap                      | Same format as Map; duplicate keys allowed.                                           |
| `.json` *(KB config)* | EmbeddedStringStorage config  | JSON object with `embedding_model`, `records_file`, and optional `index_file` fields. |
| `.kb.json`            | EmbeddedStringStorage records | JSON array of `{"id": "…", "text": "…", "metadata": {…}}` objects.                    |
| `.usearch`            | HNSW index                    | Binary file produced by the server on first load; ship it to avoid rebuild overhead.  |

Translations ship with the folder

Localized content lives in `loc/<language>/` folders beside the files they translate, so anything that copies the storage folder already carries them. If you filter what gets staged, make sure the filter keeps `**/loc/**` — a missing translation folder degrades silently to the default language rather than failing the build. See [Ship an Agent in Another Language](https://docs.tryllengine.com/how-to/ship-in-another-language.html).

## Setting `storage_data_folder`

Tell each session where the storage folder is by sending a non-empty `storage_data_folder` in `CreateSessionRequest`. Every client platform has a convenience setting:

```
client.create_session(
    inference_engine="LlamaCpp",
    storage_data_folder="/absolute/path/to/StorageData",
)
```

```
client.CreateSession(
    Tryll::InferenceEngine_LlamaCpp,
    /*allowAutoDownload=*/false,
    "MyGame",
    Tryll::InferenceEngine_Mock,
    Tryll::InferenceEngine_Mock,
    Tryll::InferenceEngine_LlamaCpp,
    "/absolute/path/to/StorageData");
```

Set **TryllRuntimeSettings → StorageDataFolder** in the Project Settings panel. The value is passed to `CreateSession` automatically when the session starts. Use a path relative to the project root, such as `Assets/StreamingAssets/TryllStringStorages` — the client resolves it to an absolute path for you, and rewrites the `Assets/StreamingAssets/` prefix onto `Application.streamingAssetsPath` in a player build. The same value therefore works in the editor and in a build with no runtime code of your own.

Set **Project Settings → Tryll → StorageDataFolder**. The value is passed to `CreateSession` automatically. In packaged builds, point this at a directory included via *Additional Non-Asset Directories to Copy* so the server executable can find the files at runtime (see [Unreal: packaging](#unreal-packaging) below).

## Platform-specific packaging

### Unity

Place the storage folder under `Assets/StreamingAssets/`:

```
Assets/
  StreamingAssets/
    StorageData/
      responses/refusal_lines.txt
      rag/adventure_quests.json
      …
```

Then set **StorageDataFolder** to the project-relative path once, in Project Settings:

```
Assets/StreamingAssets/StorageData
```

No runtime code is needed. The client absolute-ifies the value against the project root in the editor, and in a player build it rewrites the `Assets/StreamingAssets/` prefix onto `Application.streamingAssetsPath` (where Unity actually copies the folder). On all desktop Unity platforms that is a real filesystem directory the server process can read.

Keep a subfolder in the path

The build-time rewrite matches the prefix `Assets/StreamingAssets/`, including the trailing slash. A bare `Assets/StreamingAssets` does not match and resolves next to the executable instead — which works in the editor and silently breaks only in a shipped build. Always include the storage subfolder, as above.

Android / WebGL

`streamingAssetsPath` on Android is inside a `.jar` and on WebGL it requires HTTP access — neither is readable as a filesystem path by an external server process. For those targets you will need a custom extraction step or a bundled server.

### Unreal

1. Place files under `Content/StorageData/`:

```
Content/
  StorageData/
    responses/refusal_lines.txt
    rag/adventure_quests.json
```

1. In **Project Settings → Packaging → Additional Non-Asset Directories to Copy**, add `Content/StorageData`. This copies the folder verbatim into the packaged build's `Content/StorageData/` next to the executable (on Windows) so the server can read it as a filesystem path.
1. Set **Project Settings → Tryll → StorageDataFolder** to an absolute path (for shipping builds, derive it from `FPaths::ProjectContentDir()` + `"StorageData"` in a startup component or Blueprint). Avoid committing machine-specific absolute paths to source control — use a project-relative default or a per-machine override config.

!!! tip "Issue 8 — known limitation" The demo `DefaultGame.ini` currently commits a machine-specific `EditorServerExePath` and a mismatched staging path. Fix the ini before packaging by using a relative default or a per-developer override; do not commit absolute local paths.

### C++ / Python (desktop)

Ship the storage folder alongside your server executable:

```
your_app/
  tryll_server.exe
  server-config.json     # set storage_root = "StorageData"
  StorageData/
    responses/…
    rag/…
```

Set `storage_root` in `server-config.json` once (relative paths are resolved from the config file's directory):

```
{
    "storage_root": "StorageData"
}
```

With `storage_root` set, all sessions that do not override `storage_data_folder` use this directory automatically.

## Verification

To verify that path resolution is working correctly:

1. Check that `CreateAgentRequest` succeeds without `3010 StorageFileNotFound` — this confirms the server can read the file.
1. Enable `log_level: "debug"` in `server-config.json` to see path-resolution log lines at node-create time.
1. If you receive `3009 StorageOutsideRoot`, the path escapes the configured root — check that the path is relative and contains no `..` components.

## Related

- [Server Configuration → Storage root](https://docs.tryllengine.com/reference/server-configuration.html#storage-root)
- [String Storage](https://docs.tryllengine.com/reference/string-storage.html)
- [Embedded String Storage](https://docs.tryllengine.com/reference/embedded-string-storage.html)
- [Error codes 3009 / 3010 / 3011](https://docs.tryllengine.com/reference/error-codes.html#3xxx-agent)
- [Use Voice Input Hotwords](https://docs.tryllengine.com/how-to/use-voice-input-hotwords.html)
- [Concept: Lifetime and Ownership](https://docs.tryllengine.com/concepts/lifetime-and-ownership.html)
