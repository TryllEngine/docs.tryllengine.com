# Use Your Own Local Model

Register a GGUF file already on disk so Tryll can load it without a Hugging Face download.

Prerequisites

- A `.gguf` file inside the server's **process storage root**, e.g. `<Project>/TryllStorage/LocalModels/my-model-q4_k_m.gguf`. Unity and Unreal default the root to `<Project>/TryllStorage/`; a server you run yourself uses `--storage-root`, `storage_root`, or the folder of `tryll_server.exe`. See [Server Configuration → Storage root](https://docs.tryllengine.com/v0.9/reference/server-configuration.html#storage-root).
- A JSON file inside your game project, if you use the Unity or Unreal plugin. Any file name. Do **not** edit the shipped `data/models.json` — updates overwrite it.
- No engine setup. The catalog entry's `engine` decides which engine runs the model.

## Steps

1. **Put the catalog JSON somewhere inside the project.** Any file name, any folder. Same schema as `models.json`. The key fields are `name` plus one `variants` entry with `engine = "llama-cpp"`, a `local_path` **directory**, and the `files` inside it:

   ```
   {
     "models": [
       {
         "name": "My Local Model",
         "variants": [
           {
             "engine": "llama-cpp",
             "local_path": "LocalModels",
             "files": ["my-model-q4_k_m.gguf"],
             "kv_cache_type": "q8_0",
             "tool_call_support": "unsupported"
           }
         ]
       }
     ]
   }
   ```

   A relative `local_path` resolves against the process storage root, so `"LocalModels"` above means `<storage root>/LocalModels/`. An absolute path also works when it points inside that root. A path outside the root (for example `C:/models`) is ignored with a warning, and the model never resolves. A Client Prod server ignores `local_path` entirely.

   Leave the shipped `models.json` unchanged. A relative `chat_template_file` is resolved against the directory that contains your catalog file, so keep those template files inside the project. The catalog is optional — leave the setting empty if you are not adding models.

   **Unity:** Project Settings → Tryll Client → Custom Models → Catalog, or Browse.

   Auto-launch runs at `BeforeSceneLoad`, before any scene `Awake`. Subscribe before that so the first process sees the file:

   ```
   [RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.AfterAssembliesLoaded)]
   static void PointAtCatalog()
   {
       TryllRuntimeSettings.BeforeAutoLaunch += () =>
       {
           var settings = TryllRuntimeSettings.Load();
           if (!settings.TrySetUserModelsCatalogPath("Assets/Tryll/my-models.json", out string error))
               Debug.LogError(error);
       };
   }
   ```

   Calling `TrySetUserModelsCatalogPath` after the server is already running restarts it. The open client session is dropped.

   A player build copies the catalog saved in Project Settings to `Server/data/user-catalog/` next to the game, including relative chat templates. To pick a different file at runtime in a build, put it under `Assets/StreamingAssets/` and set that project-relative path from code.

   **Unreal:** Project Settings → Plugins → Tryll Client → Models → Catalog, or Browse. From code or Blueprint, `Set User Models Catalog Path`.

   The first auto-launch runs in the Tryll subsystem's `Initialize`, before BeginPlay. Set the path from your GameInstance `Init` (or from Project Settings) so the first process sees it. Calling the setter while a server is already running restarts it when Auto Launch Server is on, and the open session is dropped.

   A packaged build copies the catalog saved in Project Settings to `Server/data/user-catalog/` next to the game, including relative chat templates. A different file selected from code in a packaged game is used only when that file is already a loose file inside the project directory.

   The stored path is relative to the project and must stay inside it. If you launch `tryll_server` yourself, pass the same file as `--user-models-path`. There is no shipped user catalog. An omitted path still names a sibling `user_models.json`, and a missing file is silent. An optional `models.d` directory is loaded when it exists; the install does not create it.

1. **Validate (optional).** From the server directory, point at the file you linked:

   ```
   tryll_server --check-catalogs --user-models-path "C:/YourProject/Assets/Tryll/my-models.json"
   ```

1. **Restart the server** (or let the editor relaunch it). Your model appears in `ListModels` / the Model Manager alongside the shipped catalog.

1. **If something goes wrong:** a malformed user file is skipped and the server still starts on the shipped catalog. The startup log names the file and the fix. Duplicate names against a shipped model are ignored for that entry only (rename yours).

## Unity / Unreal

The Catalog field above is the place to link the file. Auto-launch passes it as `--user-models-path`. In a packaged game that path is the staged copy under `Server/data/user-catalog/`. A folder named `<ProjectRoot>/Tryll/models.d` is still passed as `--user-models-dir` when it exists, for extra drop-in files.

## See also

- [Model management](https://docs.tryllengine.com/v0.9/reference/model-management.html) — merge rules and catalog schema
- [Server configuration](https://docs.tryllengine.com/v0.9/reference/server-configuration.html) — `user_models_catalog_path`
