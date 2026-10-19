# Models and Inference Engines

Tryll is a model host first and a workflow engine second. This page is about the host side: what counts as a "model", how the server talks to the hardware, what the KV cache is doing under your agents, and how to think about context size and quantisation when you pick one.

## Four kinds of models

Tryll treats four kinds of models as first-class citizens:

- **Language models** — generate text. These are what [`Generate`](https://docs.tryllengine.com/v0.9/reference/nodes/generate.html) and [`ToolCall`](https://docs.tryllengine.com/v0.9/reference/nodes/tool-call.html) nodes use.
- **Embedding models** — turn text into fixed-length float vectors. [`Retrieve`](https://docs.tryllengine.com/v0.9/reference/nodes/retrieve.html) uses one to embed the current user message before searching an [embedded string storage](https://docs.tryllengine.com/v0.9/reference/embedded-string-storage.html).
- **STT models** (speech-to-text) — transcribe audio to text for voice input. Exposed in every client via the voice-input API (C++ `CreateVoiceInput`, Unity `CreateVoiceInputAsync` / `TryllVoiceInput`, Unreal `UTryllVoiceInputComponent`).
- **TTS models** (text-to-speech) — synthesise speech from text. Exposed via the [`Speak`](https://docs.tryllengine.com/v0.9/reference/nodes/speak.html) and [`GenerateAndSpeak`](https://docs.tryllengine.com/v0.9/reference/nodes/generate-and-speak.html) nodes, with ready-made playback components (`TryllSpeaker` in Unity, `UTryllSpeakerComponent` in Unreal). See [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.9/how-to/add-voice-to-an-agent.html).

Every entry in the catalog (`models.json`) declares which kind it is. Asking for a model of the wrong kind fails fast with a clear error. See [Model Management](https://docs.tryllengine.com/v0.9/reference/model-management.html) for the full catalog schema.

You do not pick an engine per kind. Each catalog entry names the engine that runs it, so downloading or registering a model is all a session needs. `CreateSession`'s `model_config` may carry an optional `engines` list of (role, engine) pairs, but only as a warm-up hint: the server starts those engines at session creation instead of at first use. It never gates a session or a load, and leaving it empty is normal. (Before protocol v13, `CreateSession` had four engine fields that defaulted to `Mock`; they are gone.)

## Inference engines: how the model actually runs

An **inference engine** is the runtime that executes the model against your CPU or GPU. Tryll currently ships these engines:

| Engine                                              | Backend                                              | Status                                                                                                                                        |
| --------------------------------------------------- | ---------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `Mock`                                              | In-process stub                                      | Used for tests; no real inference.                                                                                                            |
| `LlamaCpp`                                          | [llama.cpp](https://github.com/ggerganov/llama.cpp)  | Default for language + embedding. Also hosts SNAC TTS (Orpheus, Maya1) via a shared ONNX decoder. GGUF files; CPU + Vulkan / CUDA / ROCm GPU. |
| `SherpaOnnx`                                        | [sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx) | STT, TTS, and VAD. TTS families: Supertonic and Pocket TTS — see [TTS Models](https://docs.tryllengine.com/v0.9/reference/tts-models.html).   |
| `OnnxRuntime`                                       | ONNX Runtime                                         | TTS for the Chatterbox family — internal only, not in the public catalog.                                                                     |
| `OnnxGenAI`, `WindowsML`, `OpenVino`, `TensorRtLlm` | Reserved                                             | Enum slots exist; implementations land in future releases.                                                                                    |

The engine comes from the model's catalog entry, not from the session (see the [C++ API](https://docs.tryllengine.com/v0.9/reference/client-api/cpp/index.html#tryllclientcreatesession) or [wire protocol](https://docs.tryllengine.com/v0.9/reference/wire-protocol.html#session-lifecycle) for the optional warm-up hint).

A catalog entry lists one or more **variants**. Variants describe different files, not different engines: every variant of one model must name the same engine, and the catalog is rejected otherwise.

```
{
  "name": "Llama-3.2-3B-Instruct",
  "variants": [
    { "engine": "llama-cpp",
      "huggingface_repo": "bartowski/Llama-3.2-3B-Instruct-GGUF",
      "files": ["Llama-3.2-3B-Instruct-Q4_K_M.gguf"],
      "kv_cache_type": "q8_0",
      "tool_call_support": "supported" }
  ]
}
```

(The JSON keys are `engine`, `huggingface_repo`, and `files` — an array. Use `local_path` + `files` for a model already on disk. Every language-model variant must declare `tool_call_support` as `"supported"` or `"unsupported"`.)

The server loads a model on the engine its catalog entry names. If this server build does not include that engine, the load fails with [`6008 EngineNotAvailable`](https://docs.tryllengine.com/v0.9/reference/error-codes.html#6xxx-model-download).

## The KV cache: why generation is fast *after* the first token

A language model's prompt can be thousands of tokens long. Naively re-running the full prompt through the model on every new token would be unworkable. Instead, transformer models keep an intermediate per-token state — the **key/value cache**, or [KV cache](https://docs.tryllengine.com/v0.9/reference/glossary.html#context-kv) — that encodes "everything the model has seen so far". Adding one more token costs one more forward pass, not N.

Tryll keeps one KV cache per node, per agent. Each turn, the node compares the new prompt against the cached token sequence:

- **Unchanged prefix** is kept.
- **Stale tail** (tokens that differ) is evicted.
- **New suffix** is processed in batches.

So the cost of "remembering the conversation" is the cost of appending the new user turn's tokens — not the whole history — as long as you are not rewriting the past.

### KV cache dtype

Each llama.cpp variant can declare a `kv_cache_type`:

| Value  | Bytes/element | Notes                                                                                                |
| ------ | ------------- | ---------------------------------------------------------------------------------------------------- |
| `f16`  | 2             | Highest quality.                                                                                     |
| `q8_0` | ~1            | Default when `kv_cache_type` is absent, on every build. Indistinguishable quality for most chat use. |
| `q4_0` | ~0.5          | Halves memory vs. q8_0; small quality hit. Useful on memory-constrained GPUs.                        |

The cache size dominates VRAM / RAM for long conversations. Moving from `f16` to `q8_0` roughly halves it; from `q8_0` to `q4_0` halves it again. Catalog `kv_cache_type` is the model-load default. A node's structural `context` table can override it per context (`Inherit` keeps the catalog value). At a right-sized window (Classify / ToolCall 2048) a `q4_0` override is usually not a measurable VRAM win; the dtype delta shows up at large `n_ctx` (around 8192).

## Context size and why 4k is not 4k

Every language model has a maximum **context size** — the total number of tokens the model can attend to at once (prompt + generation buffer). Tryll reserves a slice at the end for the generation itself (`generation_reserve`), and uses the rest as the **high water mark** for projection. Tokens beyond the high water mark get trimmed by [token-budget projection](https://docs.tryllengine.com/v0.9/concepts/projection-and-token-budgets.html) before the model ever sees them.

So a model nominally advertised as "4k context" will, in practice, give you something like 3.5k of usable history — more than enough for most dialog, but worth knowing about when you design a RAG prompt.

## Embedding models

Embedding models are much smaller and simpler: a stateless encoder that turns text into fixed-dimension vectors, with mean pooling to produce one vector per input. The server calls the model once per query and once per chunk at index-build time; that is the whole story.

An embedding model is **required** whenever you create an [embedded string storage](https://docs.tryllengine.com/v0.9/reference/embedded-string-storage.html) — either sent on the wire (Path B) or declared in the `*.kb.json` config (Path A). The two sides must match.

## The loaded-model pool

Language, embedding, STT, and TTS models are loaded once per process into separate caches and reference-counted with the same retention rules:

- **Pinned** retention — set by `LoadModelRequest` for any of the four kinds. The model stays in memory until you explicitly unload it *and* no active contexts / sessions reference it.
- **OnDemand** retention (default) — set by the lazy load path when an agent, voice input, TTS, or embedding use first needs the model. The server frees unused OnDemand models after `DestroyAgentRequest` and again immediately before the next validated load into any cache (cross-cache sweep). Pinned models and models still held by agents are left alone.

VAD models appear in the catalog for download/config, but they are **not** part of this pin/unload pool — offline STT loads VAD as a dependency of the STT model.

This pair lets you either nail a model in place ("this is the chat model, always keep it warm") or let the pool breathe when sessions cycle through models. See [Lifetime and Ownership → Models](https://docs.tryllengine.com/v0.9/concepts/lifetime-and-ownership.html#models) for the full reference-counting story, including what happens when `UnloadModelRequest` arrives while agents are still using the model.

## Downloading and the "Downloaded" status

Models you did not bundle with the server can be pulled from Hugging Face via `DownloadModelRequest`. The server streams `DownloadProgress` frames and finishes with a `DownloadComplete(success=true)`. After that the model has status `Downloaded` and can be loaded. `Local` is reserved for a model whose catalog `local_path` resolved to a file on disk. See [Model Management](https://docs.tryllengine.com/v0.9/reference/model-management.html#model-status) for the full lifecycle and the distinction between on-disk statuses and `Loaded` (memory).

## Edges and pitfalls

- **No model swap mid-turn.** The model a context points to is resolved at agent-creation time. Pinning or unloading any model while an agent is running is safe: unloading *the* model a running agent depends on only drops the pin, and the model stays resident until that agent's contexts are destroyed.
- **KV cache is *per node, per agent*.** Two `Generate` nodes using the same model in the same graph still each pay their own KV cost. Budget memory accordingly.
- **Embedding and language models are not interchangeable.** Trying to point a `Retrieve` node at a language model (or a `Generate` node at an embedding model) fails validation with a clear error.
- **Context-window claims are rough.** Official context sizes assume no generation buffer; the high-water mark is what actually governs your prompt budget.
- **One base model, several behaviours.** An LLM node can run its model with [LoRA adapters](https://docs.tryllengine.com/v0.9/how-to/use-a-lora-adapter.html): small GGUF files trained for that base model. The base weights stay loaded once; only the nodes that name an adapter use it.

## Related

- [Reference: Model Management](https://docs.tryllengine.com/v0.9/reference/model-management.html)
- [Reference: Agent Parameters](https://docs.tryllengine.com/v0.9/reference/agent-parameters.html)
- [Reference: Server Configuration](https://docs.tryllengine.com/v0.9/reference/server-configuration.html)
- [Concept: Lifetime and Ownership](https://docs.tryllengine.com/v0.9/concepts/lifetime-and-ownership.html)
- [Concept: Projection and Token Budgets](https://docs.tryllengine.com/v0.9/concepts/projection-and-token-budgets.html)
- [Concept: RAG](https://docs.tryllengine.com/v0.9/concepts/rag.html)
- [How-to: Use your own local model](https://docs.tryllengine.com/v0.9/how-to/use-your-own-local-model.html)
- [How-to: Use a LoRA adapter](https://docs.tryllengine.com/v0.9/how-to/use-a-lora-adapter.html)
- [How-to: Pin and unpin models](https://docs.tryllengine.com/v0.9/how-to/pin-and-unpin-models.html)
- [How-to: Estimate memory footprint](https://docs.tryllengine.com/v0.9/how-to/estimate-memory-footprint.html)
