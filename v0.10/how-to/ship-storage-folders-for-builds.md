# Ship Storage Folders for Builds

Tryll nodes that use relative `string_storage` or `embedded_string_storage` paths need those files readable by the **server process** at runtime. The storage root is **process-wide**: the launcher sets it at startup (`--storage-root`). Sessions no longer send `storage_data_folder`.

Default for Unity and Unreal projects: **`<Project>/TryllStorage/`**, outside `Assets/` and `Content/`. See the deployment-modes contract in the contributor docs if you need every mode rule.

## File layout conventions

```
your_game/
  TryllStorage/
    responses/
      refusal_lines.txt          # CannedResponse — List kind
      polite_reject.txt
    intent/
      npc_instructions.json      # IntentToInstruction — Map kind
    rag/
      adventure_quests.json      # EmbeddedStringStorage config (Path A)
      adventure_quests.kb.json   # records file
      adventure_quests.usearch   # HNSW index, baked by the server
      adventure_quests.bm25      # keyword index, baked by the server
      adventure_quests.bake.json # what both were built from
    models/                      # Client Dev downloads land here (created if missing)
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
| `.usearch`            | HNSW index                    | Binary file the server builds from the records. Shipped games only load it.           |
| `.bm25`               | Keyword index                 | Same, for keyword and hybrid search.                                                  |
| `.bake.json`          | Bake record                   | The records file content and embedding model the two indexes were built from.         |

Translations ship with the folder

Localized content lives in `loc/<language>/` folders beside the files they translate, so anything that copies the storage folder already carries them. If you filter what gets staged, make sure the filter keeps `**/loc/**` — a missing translation folder degrades silently to the default language rather than failing the build. See [Ship an Agent in Another Language](https://docs.tryllengine.com/v0.10/how-to/ship-in-another-language.html).

## Indexes are baked when you build

A shipped game never builds an index. It loads the `.usearch`, `.bm25` and `.bake.json` beside each knowledge base config, and refuses to load a knowledge base whose files are missing or do not match its records or embedding model. The same goes for the `bpe.vocab` beside a speech model that uses [hotwords](https://docs.tryllengine.com/v0.10/how-to/use-voice-input-hotwords.html).

The Unity and Unreal plugins produce these files for you. Before staging, a Windows build runs `tryll_server --bake` on the storage folder with the editor's settings. It builds whatever is missing or out of date, and **fails the build** if any knowledge base or speech model cannot be baked, for example because its embedding model is not downloaded or its config has no `index_file`.

For a C++ or Python app, run the bake yourself before you package:

```
tryll_server --bake --mode client-dev --project-name <name> --project-dir <dir> --project-id <32 hex> --storage-root <TryllStorage>
```

Add `--check` to verify without writing anything; the exit code is non-zero when something would have to be rebuilt.

## Setting the storage root

The **server** owns the root. Clients configure the launcher; they do not send a per-session path on `CreateSession`.

Set **Project Settings → Tryll Client → Storage Data Folder**. Default `TryllStorage` (project-relative, outside `Assets/`). A path under `Assets/` is rejected. The editor launches the server with `--storage-root` pointing at that folder.

Set **Project Settings → Plugins → Tryll Client → Storage Data Folder**. Default `TryllStorage` (outside `Content/`). A path under `Content/` is rejected. The plugin passes `--storage-root` on auto-launch.

Pass `--storage-root` when you start the server, or set `storage_root` in `server-config.json`. Managed launchers expose project name, project directory, and storage root options.

## Platform-specific packaging

### Unity

Keep authored content in `<Project>/TryllStorage/` — **not** under `Assets/StreamingAssets/`. Unity would copy StreamingAssets with no model filter and would create `.meta` files for every download and derived index.

At build time `TryllServerBuildPostprocessor` copies the storage tree into `<Game>_Data/StreamingAssets/Tryll/` in the **build output**, filtering models by the Production manifest. It never writes into your project's `Assets/`.

Player storage root: `Application.streamingAssetsPath/Tryll`.

Players are unreleasable for now

Packaged players still launch the Client Dev server until a Client Prod binary ships. Do not ship those builds to end users yet.

Android / WebGL

`streamingAssetsPath` on Android is inside a `.jar` and on WebGL it requires HTTP access — neither is readable as a filesystem path by an external server process. For those targets you will need a custom extraction step or a bundled server.

### Unreal

Keep authored content in `<Project>/TryllStorage/` — **not** under `Content/`. The server cannot read Unreal `.pak` archives, and staging whole `Content/` folders has no per-model filter.

`TryllClient.Build.cs` walks the storage directory and stages each file as **NonUFS** under `$(ProjectDir)/TryllStorage/`, filtering models by the Production manifest. You do not need *Additional Non-Asset Directories to Copy* for Tryll content.

Packaged storage root: `<packaged project dir>/TryllStorage/` (project dir, not the exe folder).

Players are unreleasable for now

Same Client Dev temporary launch as Unity. Do not ship those builds to end users yet.

### C++ / Python (desktop)

Ship the storage folder beside your server config:

```
your_app/
  tryll_server.exe
  data/
    server-config.json     # optional storage_root; or pass --storage-root
  TryllStorage/
    responses/…
    rag/…
    models/…
```

```
{
    "storage_root": "../TryllStorage"
}
```

Or pass `--storage-root` on the command line (CLI wins over config).

## Verification

1. Check that `CreateAgentRequest` succeeds without `3010 StorageFileNotFound`.
1. Enable `log_level: "debug"` in `server-config.json` to see path-resolution log lines at node-create time.
1. If you receive `3009 StorageOutsideRoot`, the path escapes the configured root — check that the path is relative and contains no `..` components.

## Related

- [Server Configuration → Storage root](https://docs.tryllengine.com/v0.10/reference/server-configuration.html#storage-root)
- [String Storage](https://docs.tryllengine.com/v0.10/reference/string-storage.html)
- [Embedded String Storage](https://docs.tryllengine.com/v0.10/reference/embedded-string-storage.html)
- [Error codes 3009 / 3010 / 3011](https://docs.tryllengine.com/v0.10/reference/error-codes.html#3xxx-agent)
- [Use Voice Input Hotwords](https://docs.tryllengine.com/v0.10/how-to/use-voice-input-hotwords.html)
- [Concept: Lifetime and Ownership](https://docs.tryllengine.com/v0.10/concepts/lifetime-and-ownership.html)
