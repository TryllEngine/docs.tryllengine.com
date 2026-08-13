# Pin and Unpin Models

Keep a model warm in memory or explicitly release it. Useful when you want the first turn to be fast, or when memory pressure forces you to pick which model lives.

Prerequisites

- A connected session, with `CreateSession` already called.
- A model listed in `models.json` with status `Local` or `Downloaded` — see [Use Your Own Local Model](https://docs.tryllengine.com/how-to/use-your-own-local-model.html).

## When to pin

| Situation                                                  | Pin?                                                             |
| ---------------------------------------------------------- | ---------------------------------------------------------------- |
| Single chat model used all session long                    | **Yes** — avoids the first-turn load delay.                      |
| Many agents with different models, short-lived             | **No** — let `OnDemand` eviction recycle memory as agents close. |
| Embedding model for RAG                                    | **Yes** — embeddings run every turn; you do not want it evicted. |
| Model used only once during startup (e.g. a warm-up probe) | **No** — let it fall out when you destroy the probing agent.     |

The two [retention policies](https://docs.tryllengine.com/reference/model-management.html) apply to **language, embedding, STT, and TTS** (not VAD — VAD is download-only and rejected by `LoadModel`):

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

`UnloadModelRequest` is safe even if the model is currently in use by an active agent — the server completes the current turn first and drops the model only when the last context is gone. You get an `Ack` as soon as the unload is scheduled.

## Verify it worked

Check status with `ListModels`:

- `Status = Loaded` → model is in memory (pinned or on-demand with active users).
- `Status = Local` or `Downloaded` → model is on disk but not in memory.

Server-side log lines:

```
[info] LoadModelRequest "My Local Model"  → Pinned
[info] UnloadModelRequest "My Local Model" → freed (contexts=0)
```

If you see `freed (contexts=N)` with `N > 0`, the unload was queued behind active contexts; it will complete as soon as those agents finish.

## Common pitfalls

- **Pinning and never unpinning.** Pinned models survive session tear-downs (they are global). A long-lived server can accumulate pins if clients forget to unload. Pair every `LoadModelRequest` with a matching `UnloadModelRequest` on shutdown.
- **Expecting pin to change load cost of subsequent sessions.** Correct — pinning is global. A second client connecting will find the model already `Loaded`.
- **Trying to pin an `Absent` model.** Pin requires the model to be at least `Local` / `Downloaded`. If it is `Absent`, call `DownloadModel` first.
- **Trying to pin a VAD model.** VAD entries are catalog/download metadata only. `LoadModel` returns `ModelResolutionFailed`. Download the VAD bundle explicitly first — offline STT only resolves and loads VAD files that are already on disk (otherwise it falls back to flush-only mode).

## Related

- [Reference: Model Management](https://docs.tryllengine.com/reference/model-management.html)
- [Concept: Models and Inference Engines](https://docs.tryllengine.com/concepts/models-and-inference-engines.html)
- [Concept: Lifetime and Ownership](https://docs.tryllengine.com/concepts/lifetime-and-ownership.html)
- [How-to: Use Your Own Local Model](https://docs.tryllengine.com/how-to/use-your-own-local-model.html)
