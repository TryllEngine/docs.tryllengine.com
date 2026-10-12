# Model Management

The Tryll server treats models as first-class, named resources. Every model it can load is declared in a single catalog file, `models.json`, that the server reads at startup. Clients discover, download, load, and unload models over the [wire protocol](https://docs.tryllengine.com/v0.8/reference/wire-protocol.html); the server handles all disk I/O, HuggingFace downloads, and backend-specific loading.

This page is the reference for:

- The [`models.json`](#modelsjson) schema that tells the server what exists and how to find it.
- The [model lifecycle](#lifecycle) wire-protocol flow: `ListModels` → `DownloadModel` → `LoadModel` / `UnloadModel`.
- The [status enum](#model-status) reported per-model.
- The [retention modes](#retention-modes) that control memory occupancy.

Server-wide model settings (download directory, catalog path) live in [`server-config.json`](https://docs.tryllengine.com/v0.8/reference/server-configuration.html).

Models must be acquired explicitly

`CreateAgent`, `CreateEmbeddedStringStorage`, and `CreateVoiceInput` fail fast if a referenced model is not already on disk — the server does not download it for you. Run `ListModels` → `DownloadModel` yourself, or acquire the model from the editor Model Manager, before referencing it.

______________________________________________________________________

## `models.json` and user catalogs

Default base catalog: `data/models.json`, configurable via `models_catalog_path` in `server-config.json`. This file is **shipped by Tryll** — do not edit it; package updates overwrite it. A missing, unreadable, or invalid **base** catalog is fatal: the process does not listen.

Plugin projects point at their own catalog from Project Settings (Unity: Tryll Client → Custom Models; Unreal: Plugins → Tryll Client → Models). Any file name, any path inside the project. Empty means no extra file. The plugin passes that file as `--user-models-path`. Set it before the server's first launch, or call the setter while a server is running to restart it (the open session is dropped). A packaged game copies the saved file to `Server/data/user-catalog/` and copies relative `chat_template_file` values beside that copy. The editor uses the project file, so those relative paths resolve in place.

Running the server yourself, pass `--user-models-path` or set `user_models_catalog_path`. The default path is a sibling `user_models.json` of the base catalog. That file is not shipped; a missing file is silent. A broken user file is skipped and the server still starts; duplicate model names or shared-asset ids against the shipped catalog skip only that entry (the shipped entry wins). Extra `models.d/*.json` files are loaded the same way when that directory exists. The install does not create it. Validate with `tryll_server --check-catalogs`.

### Top-level structure

```
{
    "shared_model_assets": [
        { ... }
    ],
    "models": [
        { ... },
        { ... }
    ]
}
```

`shared_model_assets` is optional. Each asset is a file set several models share, such as the SNAC decoder: `id`, `huggingface_repo`, and `files`, each with `path`, `size_bytes` and a 64-hex-digit `sha256`. A model names an asset from its `dependencies`; every such id must resolve.

### Model descriptor fields

| Field              | Type   | Required | Description                                                                                                                                                                                                                  |
| ------------------ | ------ | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`             | string | Yes      | Human-readable model name. This is the identifier used everywhere on the wire: `CreateAgentRequest.default_model_name`, `DownloadModelRequest.model_name`, `LoadModelRequest.model_name`. Must be unique within the catalog. |
| `model_type`       | string | No       | `"embedding"`, `"stt"`, `"tts"`, or `"vad"`. **Omit for language models** — language is the implicit default.                                                                                                                |
| `audience`         | string | No       | Informational visibility hint surfaced to editor clients via `ListModels` (`"public"`, `"experimental"`, `"internal"`). Not consumed by the server.                                                                          |
| `hidden`           | bool   | No       | When `true`, the entry is omitted from `ListModels`.                                                                                                                                                                         |
| `default_sampling` | object | No       | Default sampling params for this model. Sampling params set on a node take precedence. Absent fields fall back to built-in defaults listed below. Ignored for non-language models.                                           |
| `variants`         | array  | Yes      | One entry per file set. Every variant of one model must use the same [inference engine](https://docs.tryllengine.com/v0.8/reference/glossary.html#inference-engine) — the catalog is rejected otherwise.                     |

### `default_sampling` fields

| Field               | Type   | Default | Description                                                                                           |
| ------------------- | ------ | ------- | ----------------------------------------------------------------------------------------------------- |
| `temperature`       | float  | `0.7`   | Sampling temperature; higher = more random.                                                           |
| `top_p`             | float  | `1.0`   | Nucleus sampling threshold. `1.0` disables.                                                           |
| `top_k`             | int    | `0`     | Top-K sampling cut-off. `0` disables.                                                                 |
| `min_p`             | float  | `0.05`  | Min-P filter. `0.0` disables.                                                                         |
| `repeat_penalty`    | float  | `1.0`   | Penalty for repeating tokens. `1.0` disables.                                                         |
| `presence_penalty`  | float  | `0.0`   | OpenAI-style presence penalty. `0.0` disables.                                                        |
| `frequency_penalty` | float  | `0.0`   | OpenAI-style frequency penalty. `0.0` disables.                                                       |
| `max_tokens`        | int    | `758`   | Maximum generated [tokens](https://docs.tryllengine.com/v0.8/reference/glossary.html#token) per turn. |
| `seed`              | uint32 | `0`     | RNG seed. `0` selects a random seed per turn.                                                         |

### `variants` entry fields

| Field                | Type            | Required              | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| -------------------- | --------------- | --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `engine`             | string          | Yes                   | Which [inference engine](https://docs.tryllengine.com/v0.8/reference/glossary.html#inference-engine) runs this model. Implemented engines: `"llama-cpp"` (language, embedding, and SNAC TTS), `"sherpa-onnx"` (STT / TTS / VAD) and `"onnx-runtime"` (Chatterbox TTS, internal). The server takes a model's engine from here, never from the session.                                                                                                                                                                                                                |
| `local_path`         | string          | No                    | Path to a directory containing the model file(s). Combined with `files[0]`. Takes priority over `path` and `downloads.json`. **Dev only:** must resolve inside the process storage root. Outside paths are ignored with a warning.                                                                                                                                                                                                                                                                                                                                   |
| `path`               | string          | No                    | Legacy relative path under `models_root`. Fallback when neither `local_path` nor a `downloads.json` entry resolves.                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `huggingface_repo`   | string          | No                    | HuggingFace repository slug (`"owner/repo"`). Required for downloads. Empty means the model cannot be downloaded over the wire — it must already be on disk via `local_path`.                                                                                                                                                                                                                                                                                                                                                                                        |
| `files`              | array of string | No                    | Filenames for this variant. For HuggingFace, these are downloaded from `huggingface_repo`; for local, resolved relative to `local_path`. Empty disables both download and resolution.                                                                                                                                                                                                                                                                                                                                                                                |
| `context_size`       | int             | No                    | Override the engine's default context window (in [tokens](https://docs.tryllengine.com/v0.8/reference/glossary.html#token)). `0` or absent uses the engine default — currently 8192 for `llama.cpp`.                                                                                                                                                                                                                                                                                                                                                                 |
| `kv_cache_type`      | string          | No                    | `llama.cpp` KV-cache dtype: `"f16"`, `"q8_0"`, or `"q4_0"`. Default `"q8_0"` (≈half the KV VRAM vs F16). Ignored by other engines.                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `chat_template_file` | string          | No                    | `llama.cpp` only. Path to a Jinja chat-template file that replaces the GGUF's own embedded template. Relative paths resolve against the directory of the catalog file that declares them. A packaged Unity or Unreal build copies those relative files next to the staged user catalog. Use this when a model's own GGUF conversion ships a template that doesn't render tool definitions — see [Tool Calling](https://docs.tryllengine.com/v0.8/concepts/tool-calling.html#not-every-model-can-call-tools) — even though the model itself may support tool calling. |
| `tool_call_support`  | string          | Yes (language models) | Advisory capability declaration: `"supported"` or `"unsupported"`. Required on every language-model variant (the server rejects the catalog otherwise) and not allowed on other model types. Shown in the Unity/Unreal Model Manager detail pane and returned by `ListModels`, so you can pick a tool-capable model before downloading it. Advisory only: whether a `ToolCall` node accepts the model is still decided at agent creation from the model's own chat template.                                                                                         |
| `disable_thinking`   | bool            | No                    | `llama.cpp` only. Suppresses chain-of-thought by toggling the template's `enable_thinking`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `use_jinja`          | bool            | No                    | `llama.cpp` only. `false` reverts to the legacy C-API chat-template path instead of the Jinja engine.                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `reasoning_budget`   | int             | No                    | `llama.cpp` only. Hard cap on thinking tokens.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |

**STT / TTS variant fields:**

| Field              | Type            | Required | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ------------------ | --------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `stt_family`       | string          | STT      | STT model family (e.g. `"whisper"`).                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `stt_recognizer`   | string          | No       | Recognizer sub-type when a family has more than one.                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `tts_family`       | string          | TTS      | TTS family. Sherpa-ONNX: `"supertonic"` or `"pocket"` — every other Sherpa family (VITS/Piper, Kokoro, Kitten, Matcha, ZipVoice) fails to load. llama.cpp SNAC: `"orpheus"` or `"maya1"`. ONNX Runtime: `"chatterbox"`. See [TTS Models](https://docs.tryllengine.com/v0.8/reference/tts-models.html).                                                                                                                                                                               |
| `tts_files`        | object          | TTS      | Named file map for the TTS bundle (e.g. `duration_predictor`, `text_encoder`, `vocoder`, `voice_style`, or `gguf` for SNAC) rather than the flat `files` array.                                                                                                                                                                                                                                                                                                                      |
| `tts_lang`         | string          | No       | Default synthesis language for a multilingual TTS model (e.g. `"en"`).                                                                                                                                                                                                                                                                                                                                                                                                               |
| `dependencies`     | object          | No       | Role → shared-asset id. SNAC entries declare `"snac_decoder": "snac-24khz-decoder"`.                                                                                                                                                                                                                                                                                                                                                                                                 |
| `tts_capabilities` | object          | No       | Advertised controls: `voice_description`, `delivery_instruction`, `reference_voice`, `speed`, `language_per_call`, plus `voice_presets` and `inline_tags` (each tag's `verification` is `verified` or `declared`). TTS models only. Editors show this; the loaded engine is authoritative. The shipped Supertonic and Pocket entries declare none, so `ListModels` reports them all-false even though those engines support speed, per-call language and (Pocket) a reference voice. |
| `languages`        | array of string | No       | Language codes the model supports: lowercase, 2–3 letters, no duplicates. Not allowed on language models. Drives the session-locale check (`3017`) and `ModelInfo.languages`.                                                                                                                                                                                                                                                                                                        |
| `num_threads`      | int             | No       | Thread count for the sherpa-onnx, SNAC or Chatterbox session.                                                                                                                                                                                                                                                                                                                                                                                                                        |

Memory placement is not a catalog field (an unknown key such as `memory_placement` is ignored). Set it per session with `CreateSession`'s `placement_hints`, per load with `LoadModelRequest.placement`, or with the Model Manager's **Placement** control.

Per-agent TTS knobs (`speaker_id`, `speed`, `tts_lang`, `tts_voice`, `tts_voice_description`, `tts_delivery_instruction`, `min_sentence_chars`, `first_chunk_max_chars`) are **node** params, not catalog fields — see [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.8/how-to/add-voice-to-an-agent.html).

### Minimal example — user-provided model

```
{
    "models": [
        {
            "name": "My Local Model",
            "variants": [
                {
                    "engine":            "llama-cpp",
                    "local_path":        "models/my-local-gguf",
                    "files":             ["my-model.gguf"],
                    "tool_call_support": "unsupported"
                }
            ]
        }
    ]
}
```

### Downloadable example — HuggingFace + tuned sampling

```
{
    "models": [
        {
            "name": "Llama 3.2 3B Instruct (Q4_K_M)",
            "default_sampling": {
                "temperature":    0.6,
                "top_p":          0.9,
                "top_k":          50,
                "min_p":          0.05,
                "repeat_penalty": 1.2
            },
            "variants": [
                {
                    "engine":            "llama-cpp",
                    "huggingface_repo":  "bartowski/Llama-3.2-3B-Instruct-GGUF",
                    "files":             ["Llama-3.2-3B-Instruct-Q4_K_M.gguf"],
                    "tool_call_support": "supported"
                }
            ]
        }
    ]
}
```

### Embedding models

Models with `"model_type": "embedding"` are declared the same way and referenced by name from [embedded string storages](https://docs.tryllengine.com/v0.8/reference/embedded-string-storage.html) and from [Retrieve node](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve.html) params. Their sampling fields are ignored; only the file resolution and engine fields matter.

______________________________________________________________________

## Lifecycle

The wire-protocol flow for putting a model under a running agent:

```
sequenceDiagram
    participant C as Client
    participant S as Server
    C->>S: ListModelsRequest
    S-->>C: ListModelsResponse([ModelInfo])
    alt status == Absent
        C->>S: DownloadModelRequest(name)
        S-->>C: DownloadProgress × N
        S-->>C: DownloadComplete(success=true)
    end
    C->>S: LoadModelRequest(name)
    S-->>C: LoadModelResponse
    C->>S: CreateAgentRequest(default_model_name=name, ...)
    S-->>C: CreateAgentResponse(agent_id)
    note over C,S: Agent runs turns...
    C->>S: UnloadModelRequest(name)
    S-->>C: Ack
```

Key rules:

- **You do not have to call `LoadModelRequest` explicitly.** An agent whose graph references a not-yet-loaded language / embedding / STT / TTS model triggers on-demand load at `CreateAgent` (or voice-input / storage creation) time.
- **`LoadModelRequest` pins the model** for any of the four managed kinds. The server keeps it resident until `UnloadModelRequest`, regardless of agent count. Pinning is process-global.
- **`UnloadModelRequest` on a shared model is polite.** If any agent still uses the model, it stays resident; it is freed when the last user goes away.
- **VAD is not pin/unloadable.** Catalog `vad` entries are download / config metadata; offline STT loads them as a dependency. Sending `LoadModelRequest` for a VAD model returns `ModelResolutionFailed`.
- **Unused OnDemand models are swept before the next load.** When the server is about to load a new model into any of the four caches, it first frees idle OnDemand entries across all caches (pinned models and models still held by agents are left alone).
- **`DeleteModelRequest` removes the downloaded files from disk.** It reverses a download (back to `Absent`); it does not touch user-supplied `local_path` models.

See [How to pin and unpin models](https://docs.tryllengine.com/v0.8/how-to/pin-and-unpin-models.html) for the end-to-end walkthrough.

### `ModelInfo` fields

`ListModelsResponse` carries one `ModelInfo` per catalog entry, enriched with the metadata editors and clients need:

| Field               | Description                                                                                                    |
| ------------------- | -------------------------------------------------------------------------------------------------------------- |
| `name`              | Catalog name — the identifier used everywhere on the wire.                                                     |
| `status`            | Where the model is now (see [Model status](#model-status)).                                                    |
| `huggingface_repo`  | Download source slug, or empty if not downloadable.                                                            |
| `size_bytes`        | On-disk size when present.                                                                                     |
| `model_type`        | `"language"`                                                                                                   |
| `engine`            | `"llama-cpp"`                                                                                                  |
| `audience`          | `"public"`                                                                                                     |
| `tool_call_support` | Advisory tool-calling capability from the catalog: `"supported"`                                               |
| `context_size`      | Context window in tokens; `0` = engine default.                                                                |
| `files`             | Resolved variant filenames.                                                                                    |
| `local_path`        | On-disk directory, or empty when not on disk.                                                                  |
| `default_sampling`  | The catalog's recommended sampling params, if any.                                                             |
| `languages`         | Catalog-declared language codes; empty means undeclared, not "none".                                           |
| `dependencies`      | Shared assets the model needs: `id`, `role`, `status`, `size_bytes`, `folder`, `local_path`, `failure_reason`. |
| `tts_capabilities`  | TTS models only: the catalog's advertised controls, voice presets and inline tags.                             |

______________________________________________________________________

## Model status

The `ModelInfo.status` field in `ListModelsResponse` reports where a model is right now:

| Value         | Int | Meaning                                                                                  |
| ------------- | --- | ---------------------------------------------------------------------------------------- |
| `Absent`      | 0   | Known in catalog but not on disk and not downloading.                                    |
| `Local`       | 1   | User-provided path resolved (`local_path`); on disk; can be loaded.                      |
| `Downloading` | 2   | Transfer in progress. `DownloadProgress` frames are flowing.                             |
| `Loaded`      | 3   | Currently resident in memory for its model kind (language, STT, TTS, or embedding).      |
| `Downloaded`  | 4   | On disk from a HuggingFace download; can be loaded.                                      |
| `Incomplete`  | 5   | Catalog parent is present but a shared-asset dependency is missing or still downloading. |

Ordinals match the `ModelStatus` enum on the [wire protocol](https://docs.tryllengine.com/v0.8/reference/wire-protocol.html) and `ETryllModelStatus` in the Unreal client.

Loaded is per model kind

`Loaded` means the model is resident in the server's in-memory cache for **its specific kind**. A language model and an STT model are tracked independently — a voice-input session that pins an STT model will show it as `Loaded` whatever other models are loaded. `ListModels` reports every catalog model with its own engine and status, whatever engines the session hinted at.

______________________________________________________________________

## Retention modes

Tryll decides what to keep in memory based on a simple rule. The same two modes apply to **language, embedding, STT, and TTS** (each kind has its own cache; sweeps are cross-cache):

| Retention    | How it is triggered                                                                                                 | When the model is freed                                                                                   |
| ------------ | ------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| **Pinned**   | `LoadModelRequest`                                                                                                  | On `UnloadModelRequest` (delayed until no agent uses it).                                                 |
| **OnDemand** | Implicit: an agent's graph (or voice input / TTS / embedding use) references the model, no prior `LoadModelRequest` | When the last user is destroyed **or** when the next validated model load sweeps unused OnDemand entries. |

Use **Pinned** for the hot model(s) on the machine — the latency cost of the first load is paid once at startup. Use **OnDemand** for secondary models where memory matters more than first-turn latency.

These labels match the client-facing names used in the glossary ([pinned retention](https://docs.tryllengine.com/v0.8/reference/glossary.html#pinned-retention), [on-demand retention](https://docs.tryllengine.com/v0.8/reference/glossary.html#ondemand-retention)).

See [Lifetime and Ownership → Models](https://docs.tryllengine.com/v0.8/concepts/lifetime-and-ownership.html#models) for the reference-counting story that connects Pinned / OnDemand to what nodes inside an agent actually hold.

______________________________________________________________________

## Editor window

Both plugins ship an editor-only **Model Manager** that drives the lifecycle above through a GUI — browse the catalog, download / load / unload / delete, and tag models for your build. Open it from **Window → Tryll → Model Manager** (Unity) or **Window → Tryll → Tryll Model Manager** (Unreal). The task walkthrough is [Manage Models in the Editor](https://docs.tryllengine.com/v0.8/how-to/manage-models-in-editor.html); this section is the reference for the two pieces of editor state it adds.

### Build registration

Each model can be tagged with a registration tier that decides whether it ships in a packaged build:

| Tier                    | Int | Meaning                                                 |
| ----------------------- | --- | ------------------------------------------------------- |
| `None` / Not registered | 0   | Not part of the build.                                  |
| `Experimental`          | 1   | Available in the editor; stripped from packaged builds. |
| `Production`            | 2   | Shipped in the packaged build.                          |

The registration is **project-side**, not server-side:

- **Unity:** a `TryllModelManifest` ScriptableObject at `Assets/Resources/TryllModelManifest.asset` (it must sit directly in a `Resources` folder so player builds can load it).
- **Unreal:** the `Tryll Model Manifest` developer settings, persisted to `DefaultGame.ini`.

### Registered-model pickers — `(tryll_model)`

Model-name fields on nodes and components render as a dropdown of registered models of the matching kind rather than a free-text box. This is driven by the `(tryll_model: "...")` schema attribute on the field; the editor resolves it against the manifest above:

| Field                                                                           | Kind       |
| ------------------------------------------------------------------------------- | ---------- |
| `Generate` / `ToolCall` / `ClassifyIntentLLM` / `GenerateAndSpeak` `model_name` | `language` |
| `GenerateAndSpeak` / `Speak` `tts_model_name`                                   | `tts`      |
| Voice Input component STT model                                                 | `stt`      |

Unity emits a `[TryllModelType("...")]` drawer; Unreal swaps the field to `FTryllModelName` with `meta=(TryllModelType="...")`. A value that is set but unregistered still displays (never silently dropped), but only `Production`-tagged models ship in a build. The wire still carries the model name as a plain string — the picker is editor sugar.

______________________________________________________________________

## Download record — `downloads.json`

The server maintains a `downloads.json` file inside `models_root` — its download ledger — to track completed HuggingFace downloads. This file is **server-managed**; do not edit it by hand. Its contents feed `ModelInfo.status == Downloaded` at startup so that a previously downloaded model can be loaded without re-downloading. Shared model assets (for example the SNAC decoder) live in a sibling `assets` array with their own records. The server stores a per-file SHA-256 on each asset (`file_sha256`) when the download finishes. Listing compares that digest to the catalog pin; the file is re-hashed when a model actually loads the asset. Deleting a parent model does not delete a shared asset. A parent whose asset is still missing, the wrong size, or pinned to a different digest reports `Incomplete`. Release staging copies each reachable asset once.

Structure (for reference only):

```
{
    "entries": [
        {
            "name":        "Llama 3.2 3B Instruct (Q4_K_M)",
            "engine":      "llama-cpp",
            "folder":      "bartowski--Llama-3.2-3B-Instruct-GGUF",
            "files":       ["Llama-3.2-3B-Instruct-Q4_K_M.gguf"],
            "total_bytes": 2019377152
        }
    ]
}
```

`engine` is stored as the same engine name the catalog uses (`"llama-cpp"`, `"sherpa-onnx"`, `"onnx-runtime"`), not as the `InferenceEngine` ordinal the [wire protocol](https://docs.tryllengine.com/v0.8/reference/wire-protocol.html) carries.

______________________________________________________________________

## Errors

| Code                                                                                       | Cause                                                                                                                                              |
| ------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`6001`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#6xxx-model-download) | Download failed (HTTP, interrupted, checksum).                                                                                                     |
| [`6002`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#6xxx-model-download) | `DownloadNotAvailable` — the model has no `huggingface_repo`/`files` configured, so there is nothing to download (e.g. a `local_path`-only entry). |
| [`6003`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#6xxx-model-download) | Disk full in the download directory.                                                                                                               |
| [`6004`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#6xxx-model-download) | A download for this model is already active.                                                                                                       |
| [`6005`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#6xxx-model-download) | `ModelResolutionFailed` — the model is not in the catalog, or it failed to load. Also returned by `LoadModel` for a VAD-only entry.                |
| [`6008`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#6xxx-model-download) | `EngineNotAvailable` — the model's catalog engine is not included in this server build.                                                            |

______________________________________________________________________

## Client bindings

- **C++:** `Tryll::Client::TryllClient::ListModels` / `DownloadModel` / `LoadModel` / `UnloadModel` — `TryllClient.h`
- **Python:** `tryll_client.TryllClient.list_models` / `download_model` / `load_model` / `unload_model` — `client.py`
- **Unreal:** `UTryllSubsystem::RequestListModels` / `RequestDownloadModel` / `RequestLoadModel` / `RequestUnloadModel` — `TryllSubsystem.h`

## Related

- [Server Configuration](https://docs.tryllengine.com/v0.8/reference/server-configuration.html) — server-wide settings that govern model storage.
- [Concept: Models and inference engines](https://docs.tryllengine.com/v0.8/concepts/models-and-inference-engines.html)
- [How to use your own local model](https://docs.tryllengine.com/v0.8/how-to/use-your-own-local-model.html)
- [How to pin and unpin models](https://docs.tryllengine.com/v0.8/how-to/pin-and-unpin-models.html)
- [Agent Parameters](https://docs.tryllengine.com/v0.8/reference/agent-parameters.html) — how agents reference models.
- [Glossary](https://docs.tryllengine.com/v0.8/reference/glossary.html#model-catalog)
