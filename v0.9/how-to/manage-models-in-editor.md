# Manage Models in the Editor

Both the Unity and Unreal plugins include an editor-only **Model Manager** window for working with the server's model catalog without writing any wire code: browse every model the server knows about, download and load them, free disk space, and mark which models ship in your packaged build.

It is a graphical front end over the model-management [wire protocol](https://docs.tryllengine.com/v0.9/reference/wire-protocol.html) (`ListModels`, `DownloadModel`, `LoadModel` / `UnloadModel`, `DeleteModel`). The catalog itself is defined server-side in [`models.json`](https://docs.tryllengine.com/v0.9/reference/model-management.html#modelsjson).

Prerequisites

- The Tryll plugin installed (see [First Inference in Unity](https://docs.tryllengine.com/v0.9/getting-started/first-inference-unity.html) / [Unreal](https://docs.tryllengine.com/v0.9/getting-started/first-inference-unreal.html)).
- A `TryllRuntimeSettings` asset (Unity) or configured plugin settings (Unreal). The window launches the bundled server automatically, the same way the [Chat Window](https://docs.tryllengine.com/v0.9/how-to/test-agent-in-editor.html) does.

## Step 1 — open the window

**Window → Tryll → Model Manager**

The window opens with the title **Tryll Models** and connects to the server automatically.

**Window → Tryll → Tryll Model Manager**

Opens as a dockable tab (**Tryll Model Manager**) and connects on open.

The layout is the same in both engines: a **filters** column on the left, the **model list** in the middle, and a **properties + actions** panel on the right.

Model Manager with a downloaded language model selected. The pills in the list show build registration (**PROD**, **EXP**, or not registered); the dot colour is the model status from the legend at the bottom. Unity shown; the Unreal panel has the same layout plus a search box and an Audience filter.

## Step 2 — find a model

Use the left column to narrow the list:

| Filter       | Values                                          |
| ------------ | ----------------------------------------------- |
| Search       | Match by model name or HuggingFace repo.        |
| Model type   | Language, Embedding, STT, TTS, VAD.             |
| Engine       | `llama-cpp`, `sherpa-onnx`, `onnx-runtime`.     |
| Status       | Absent, Downloaded, Local, Loaded, Downloading. |
| Registration | Production, Experimental, Not registered.       |

The left column also totals how many models are registered for the build and the on-disk / shipped size, so you can see your build's model footprint at a glance. Select a row to populate the properties panel on the right.

## Step 3 — download, load, unload, delete

Select a model, then use the action buttons in the properties panel:

| Action           | Effect                                                                                                                                               | Wire call       |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | --------------- |
| **Download**     | Fetch the model's files from HuggingFace; a progress bar tracks bytes.                                                                               | `DownloadModel` |
| **Load**         | Pin a language / embedding / STT / TTS model into server memory so the first turn has no load latency. VAD cannot be pinned (catalog/download only). | `LoadModel`     |
| **Unload**       | Unpin a loaded managed model (freed once no agent still uses it).                                                                                    | `UnloadModel`   |
| **Delete files** | Remove the downloaded files from disk.                                                                                                               | `DeleteModel`   |

A toolbar **Download all** action fetches every still-absent model in the current filtered list. See [Model Management → Lifecycle](https://docs.tryllengine.com/v0.9/reference/model-management.html#lifecycle) for the load/unload rules (you usually do not need to **Load** manually — an agent loads its model on demand at `CreateAgent`).

## Step 4 — register models for your build

Each model can be tagged with a **build registration** tier, in the properties panel:

| Tier               | Meaning                                                 |
| ------------------ | ------------------------------------------------------- |
| **Production**     | Shipped in your packaged build.                         |
| **Experimental**   | Available in the editor; stripped from packaged builds. |
| **Not registered** | Not part of the build at all.                           |

Registrations are stored in your project, not on the server:

Persisted to a `TryllModelManifest` asset that the Model Manager writes at `Assets/Resources/TryllModelManifest.asset`. It must stay in a `Resources` folder so a player build can load it.

Persisted to `DefaultGame.ini` via the `Tryll Model Manifest` developer settings.

## Step 5 — pick registered models on nodes

Model-name fields elsewhere in the editor read this manifest. A `Generate` node's **model**, `GenerateAndSpeak`'s **TTS model**, and the Voice Input component's **STT model** all show a dropdown filtered to registered models of the right kind, with a tier badge next to each. A name you set that is not registered still displays — it is never silently dropped — but it will not ship unless you register it Production.

This is the editor side of the `(tryll_model)` schema attribute; see [Model Management → Build registration](https://docs.tryllengine.com/v0.9/reference/model-management.html#editor-window).

## Common problems

**The list is empty / "Connecting…" never resolves** : The server did not start or did not bind port 9100. Check the Unity Console or Unreal Output Log for `[Tryll Server]` lines, as in the [Chat Window troubleshooting](https://docs.tryllengine.com/v0.9/how-to/test-agent-in-editor.html#common-problems).

**A model shows `Absent` and Download is disabled** : The catalog entry has no `huggingface_repo`, so it cannot be fetched over the wire — it must be supplied on disk. See [Use Your Own Local Model](https://docs.tryllengine.com/v0.9/how-to/use-your-own-local-model.html).

**A registered model is missing from a build** : Only **Production** models ship. Experimental and unregistered models are excluded from packaged builds by design.

## Related

- [Reference: Model Management](https://docs.tryllengine.com/v0.9/reference/model-management.html)
- [How-to: Use Your Own Local Model](https://docs.tryllengine.com/v0.9/how-to/use-your-own-local-model.html)
- [How-to: Inspect Server Memory](https://docs.tryllengine.com/v0.9/how-to/inspect-server-memory.html) — see load/unload change the live RAM/VRAM snapshot
- [How-to: Pin and Unpin Models](https://docs.tryllengine.com/v0.9/how-to/pin-and-unpin-models.html)
- [How-to: Verify Required Models Before Play](https://docs.tryllengine.com/v0.9/how-to/verify-models-before-play.html) — the editor guard that catches a registered-but-undownloaded model before Play/PIE or Tryll Chat.
- [Concept: Models and inference engines](https://docs.tryllengine.com/v0.9/concepts/models-and-inference-engines.html)
