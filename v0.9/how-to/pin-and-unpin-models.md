# Pin and Unpin Models

Keep a model warm in memory or explicitly release it. Useful when you want the first turn to be fast, or when memory pressure forces you to pick which model lives.

Prerequisites

- A connected session, with `CreateSession` already called.
- A model listed in `models.json` with status `Local` or `Downloaded` — see [Use Your Own Local Model](https://docs.tryllengine.com/v0.9/how-to/use-your-own-local-model.html).

## When to pin

| Situation                                                  | Pin?                                                             |
| ---------------------------------------------------------- | ---------------------------------------------------------------- |
| Single chat model used all session long                    | **Yes** — avoids the first-turn load delay.                      |
| Many agents with different models, short-lived             | **No** — let `OnDemand` eviction recycle memory as agents close. |
| Embedding model for RAG                                    | **Yes** — embeddings run every turn; you do not want it evicted. |
| Model used only once during startup (e.g. a warm-up probe) | **No** — let it fall out when you destroy the probing agent.     |

The two [retention policies](https://docs.tryllengine.com/v0.9/reference/model-management.html) apply to **language, embedding, STT, and TTS** (not VAD — VAD is download-only and rejected by `LoadModel`):

- **`Pinned`** — set by `LoadModelRequest` / Blueprint **Request Load Model**. Only unloaded when you send `UnloadModelRequest` **and** no active contexts still reference the model.
- **`OnDemand`** — the default. Set implicitly when you create an agent (or voice input / TTS / embedding use) and the model is not already loaded. The server frees unused OnDemand models after `DestroyAgent`, and again immediately before the next validated model load (across all four caches).

## Steps — pin a model

```
TryllClient.Instance.LoadModelComplete += (modelName, success) => {
    if (success) Debug.Log($"{modelName} pinned");
    else         Debug.LogError($"Failed to pin {modelName}");
};
TryllClient.Instance.RequestLoadModel("My Local Model");
```

Call **Request Load Model** on `UTryllSubsystem` with the model name. Bind **On Load Model Complete** to wait for `bSuccess = true`.

```
try {
    client.LoadModel("My Local Model");
} catch (const Tryll::Client::TryllError& ex) {
    std::cerr << "load failed: " << ex.what() << "\n";
}
```

```
client.load_model("My Local Model")   # returns when load completes
```

After a successful pin, the model stays in memory across agent create / destroy cycles until you explicitly unload it.

## Steps — unpin a model

```
TryllClient.Instance.UnloadModelComplete += (modelName, success) =>
    Debug.Log($"{modelName} unloaded: {success}");
TryllClient.Instance.RequestUnloadModel("My Local Model");
```

Call **Request Unload Model** on `UTryllSubsystem` with the model name. Bind **On Unload Model Complete**.

```
client.UnloadModel("My Local Model");
```

```
client.unload_model("My Local Model")
```

`UnloadModelRequest` is safe even if the model is currently in use by an active agent. The server drops the pin and sends `Ack` immediately. A model that no context holds is freed at once. A model that agents still hold stays resident until those agents' contexts are destroyed (for example by `DestroyAgent`); finishing a turn does not free it.

## Verify it worked

Check status with `ListModels`:

- `Status = Loaded` → model is in memory (pinned or on-demand with active users).
- `Status = Local` or `Downloaded` → model is on disk but not in memory.

Server-side log lines:

```
[Session 1] LoadModel requested: 'My Local Model'
[ModelManager] Pinned 'My Local Model'
[Session 1] UnloadModel requested: 'My Local Model'
[ModelManager] Unloaded 'My Local Model'
[Session 1] Model 'My Local Model' unloaded immediately
```

If the last line reads `unload deferred (active contexts)` instead, agents still hold the model. It stays resident until their contexts are destroyed.

## Common pitfalls

- **Pinning and never unpinning.** Pinned models survive session tear-downs (they are global). A long-lived server can accumulate pins if clients forget to unload. Pair every `LoadModelRequest` with a matching `UnloadModelRequest` on shutdown.
- **Expecting pin to change load cost of subsequent sessions.** Correct — pinning is global. A second client connecting will find the model already `Loaded`.
- **Trying to pin an `Absent` model.** Pin requires the model to be at least `Local` / `Downloaded`. If it is `Absent`, call `DownloadModel` first.
- **Trying to pin a VAD model.** VAD entries are catalog/download metadata only. `LoadModel` returns `ModelResolutionFailed`. Download the VAD bundle explicitly first — offline STT only resolves and loads VAD files that are already on disk (otherwise it falls back to flush-only mode).

## Related

- [Reference: Model Management](https://docs.tryllengine.com/v0.9/reference/model-management.html)
- [Concept: Models and Inference Engines](https://docs.tryllengine.com/v0.9/concepts/models-and-inference-engines.html)
- [Concept: Lifetime and Ownership](https://docs.tryllengine.com/v0.9/concepts/lifetime-and-ownership.html)
- [How-to: Use Your Own Local Model](https://docs.tryllengine.com/v0.9/how-to/use-your-own-local-model.html)
