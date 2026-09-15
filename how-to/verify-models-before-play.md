# Verify Required Models Before Play

Both editor plugins register models per-project as **Production** or **Experimental** in the [Model Manager](https://docs.tryllengine.com/how-to/manage-models-in-editor.html), then download them there. If a teammate pulls a repo change that adds or replaces a registered model, that download step is easy to miss — the next time you press Play (or open the Tryll Chat window) with a stale local model cache, `CreateAgent` used to fail with a `GraphCompilationFailed` error that gave no hint a download was the fix.

The editor guard closes that gap. It runs automatically at every point an agent is about to be created in the editor. The automatic **Play / PIE** check is on by default and can be turned off per project if you prefer to manage model provisioning yourself — see [Turning off the Play/PIE check](#turning-off-the-playpie-check).

## What it checks

On each guarded entry point, the guard loads your project's model manifest (the same Production/Experimental registrations you set in the [Model Manager](https://docs.tryllengine.com/how-to/manage-models-in-editor.html)), compares every registered model of every kind (language, embedding, STT, TTS, VAD) against the live `ListModels` status, and treats anything `Absent` as needing attention.

## Where it runs

| Entry point                         | Behaviour                                                                                                                                                                                                                    |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Play** (Unity) / **PIE** (Unreal) | Blocks automatically: downloads every missing registered model behind a cancelable progress dialog, then continues into Play/PIE once ready. Cancelling, or a model that fails to download, keeps you in the editor instead. |
| **Tryll Chat window/panel**         | Same automatic, blocking treatment as Play — the guard runs right after the session is created and before the chat agent is built.                                                                                           |
| **Model Manager window/panel**      | Lighter-touch: after the model list loads, if anything registered is missing you get a Yes/No prompt instead of an automatic download — this is also where you'd go to fix it manually.                                      |

Unity: what a cancelled Play looks like

Unity has no way to pause entering Play mid-transition, so the guard cancels the transition, runs the check with a progress dialog, and — once every model is confirmed downloaded — automatically re-enters Play for you. You don't need to press Play twice.

## Turning off the Play/PIE check

The automatic **Play (Unity) / PIE (Unreal)** check is on by default but can be disabled per project:

- **Unreal:** Project Settings → Plugins → **Tryll Client** → *Editor* → **Enable Play Mode Model Guard** (the `bEnablePlayModeModelGuard` field on `UTryllRuntimeSettings`, saved to `DefaultGame.ini`). Uncheck it to enter PIE immediately with no pre-flight model check.
- **Unity:** Project Settings → **Tryll Client** → *Play Mode (Editor)* → **Enable Play Mode Model Guard** (the `EnablePlayModeModelGuard` field on `TryllRuntimeSettings`). Uncheck it to enter Play immediately with no pre-flight model check.

The toggle affects **only** the automatic Play/PIE check — the Tryll Chat window/panel still verifies models before creating its agent, and the Model Manager still prompts for missing ones. With the check off, a required model that isn't on disk surfaces as a `CreateAgent` failure at runtime instead of being caught (and offered for download) before Play.

## A model that can't be downloaded

Some registered models are supplied as a local file path rather than a HuggingFace repo (see [Use Your Own Local Model](https://docs.tryllengine.com/how-to/use-your-own-local-model.html)). If one of those is registered but missing on disk, the guard cannot fetch it — it surfaces a clear error naming the model instead of attempting a download. Provide the file or remove the registration in the [Model Manager](https://docs.tryllengine.com/how-to/manage-models-in-editor.html).

## Relationship to auto model downloading

Earlier releases had a server-side `allow_auto_model_downloading` session flag that made the *server* silently fetch a model mid-`CreateAgent`. That flag has been removed — model registration is now the only way to acquire models deliberately. This guard is the editor-side replacement: it catches the same "model referenced but never downloaded" case, earlier, and with a clear prompt instead of a silent multi-gigabyte download hidden inside a normal request.

Editor-only in both engines — packaged/Standalone builds never run this code, since they bundle Production models at build time and never download at runtime.

## Related

- [How-to: Manage Models in the Editor](https://docs.tryllengine.com/how-to/manage-models-in-editor.html) — where registrations live and how to fix a missing model manually.
- [How-to: Use Your Own Local Model](https://docs.tryllengine.com/how-to/use-your-own-local-model.html) — non-downloadable models.
