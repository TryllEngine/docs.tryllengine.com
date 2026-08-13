# Estimate the RAM and VRAM Your Setup Needs

Work out, before you ship, roughly how much **system RAM** and **GPU VRAM** a given model line-up will use — so you can pick models that fit your target hardware and avoid out-of-memory surprises on a player's machine.

Prerequisites

- The models you plan to use, listed in `models.json` (each has a download size you can read — see [Use Your Own Local Model](https://docs.tryllengine.com/how-to/use-your-own-local-model.html)).
- A rough idea of your context length (how long conversations get).

## What uses RAM vs. VRAM

With GPU offload on (the default), the four things that cost memory split cleanly:

| What                                | RAM (system)         | VRAM (GPU)                          | Scales with             |
| ----------------------------------- | -------------------- | ----------------------------------- | ----------------------- |
| **Language model weights**          | ≈ download size      | ≈ download size                     | model file size         |
| **KV cache** (conversation context) | negligible           | **large — hundreds of MB to ~2 GB** | context length × model  |
| **Embedding model** (for RAG)       | tens of MB           | tens of MB                          | model file size (small) |
| **STT / TTS** (voice in/out)        | ≈ 1–2× download size | **none**                            | model file size         |

Two facts do most of the work:

- **Weights cost the same in both RAM and VRAM.** A GGUF model is memory-mapped (counts as RAM) *and* offloaded to the GPU (counts as VRAM), so a 4 GB model occupies ~4 GB of *each*. See [Models and Inference Engines](https://docs.tryllengine.com/concepts/models-and-inference-engines.html).
- **Voice models live in RAM only.** Speech-to-text and text-to-speech run on the CPU, so they add **zero VRAM** — only system RAM.

## The recipe

**VRAM** ≈ (sum of language + embedding model download sizes) + a **KV-cache allowance**.

- KV allowance is the part that grows with conversation length. As a rough guide on a Q4 model: budget **~0.7 GB** for short/RAG-style turns and up to **~1.5–2 GB** for long multi-turn dialogs. Each `Generate` node with its own model pays its own KV — count them separately.

**RAM** ≈ (language model download size) + (STT size + TTS size, each ×~1.5 if you use voice) + a few hundred MB of runtime overhead.

- The language model's weights show up in *both* budgets (mmap); that is expected, not double-counting a leak.

Look up each model's **download size** — the table in [Built-in model sizes](#built-in-model-sizes) below lists the measured sizes. (The raw numbers come from the `downloads.json` manifest. The editor **Model Manager** window also surfaces each model's size once it is downloaded — see [Manage Models in the Editor](https://docs.tryllengine.com/how-to/manage-models-in-editor.html); the catalog file `models.json` lists *which* models exist but not their size.) That download size is the number you plug in.

## Built-in model sizes

Measured on-disk sizes of the downloaded models, taken from the `downloads.json` manifest (exact bytes, rounded here). Use these as the per-model number in the recipe above. Language and embedding sizes count toward **both** RAM and VRAM; STT and TTS sizes count toward **RAM only**.

### Language models (Q4_K_M)

| Model                              | Download size |
| ---------------------------------- | ------------- |
| Gemma 3 1B Instruct                | 806 MB        |
| Llama 3.2 1B Instruct              | 808 MB        |
| SmolLM2 1.7B Instruct              | 1.1 GB        |
| Qwen 3.5 2B                        | 1.3 GB        |
| Granite 4.0 H-Micro Instruct       | 1.9 GB        |
| Llama 3.2 3B Instruct              | 2.0 GB        |
| Ministral 3 3B Instruct            | 2.1 GB        |
| Gemma 3 4B Instruct                | 2.5 GB        |
| Phi-4 Mini Instruct                | 2.5 GB        |
| Qwen 3.5 4B                        | 2.7 GB        |
| NVIDIA Nemotron 3 Nano 4B          | 2.9 GB        |
| Granite 4.0 H-Tiny Instruct        | 4.3 GB        |
| Mistral 7B Instruct                | 4.4 GB        |
| Llama 3.1 8B Instruct              | 4.9 GB        |
| Llama 3.1 Nemotron Nano 8B v1      | 4.9 GB        |
| Llama 3.1 Nemotron 8B UltraLong 1M | 4.9 GB        |
| NVIDIA Orchestrator 8B             | 5.0 GB        |
| Ministral 3 8B Instruct            | 5.2 GB        |
| Gemma 4 E4B Instruct               | 5.4 GB        |

### Embedding models (Q4_K_M)

| Model                 | Download size |
| --------------------- | ------------- |
| All-MiniLM-L6-v2      | 21 MB         |
| BGE Small EN v1.5     | 25 MB         |
| Nomic Embed Text v1.5 | 84 MB         |

### Speech-to-text (STT, int8)

| Model                                | Download size |
| ------------------------------------ | ------------- |
| Streaming Zipformer EN               | 73 MB         |
| Whisper Tiny EN                      | 104 MB        |
| Whisper Base EN                      | 161 MB        |
| Streaming Paraformer Bilingual zh-en | 237 MB        |
| SenseVoice Multilingual              | 237 MB        |
| Whisper Small EN                     | 376 MB        |
| Parakeet TDT 0.6B v2                 | 661 MB        |
| Parakeet TDT 0.6B v3                 | 670 MB        |
| Whisper Medium EN                    | 946 MB        |
| Whisper Large v3 Turbo               | 1.0 GB        |

### Text-to-speech (TTS) and VAD

| Model               | Download size |
| ------------------- | ------------- |
| Silero VAD          | 2 MB          |
| Supertonic 3 (int8) | 145 MB        |
| Pocket TTS (int8)   | 200 MB        |

RAM/VRAM reminder

For STT/TTS, the *runtime* RAM is roughly 1–2× the download size (the speech runtime allocates working buffers on top of the model file). For language and embedding models, the weights occupy ≈ the download size in **each** of RAM and VRAM (mmap + GPU offload).

## Worked example

A voice NPC: **Llama 3.1 8B** (≈4.9 GB) for chat, **All-MiniLM-L6-v2** (≈21 MB) for RAG retrieval, and **Supertonic 3 (int8)** (≈145 MB) for speech output, with moderately long conversations.

| Budget   | Calculation                                                  | Total        |
| -------- | ------------------------------------------------------------ | ------------ |
| **VRAM** | 4.9 GB (weights) + 0.02 GB (embedding) + ~1.0 GB (KV)        | **≈ 5.9 GB** |
| **RAM**  | 4.9 GB (weights, mmap) + ~0.3 GB (Supertonic TTS) + overhead | **≈ 5.5 GB** |

So this NPC wants a **6 GB+ GPU** and comfortably runs in **8 GB system RAM**. Swapping to a 3B model (≈2.0 GB) drops the VRAM need to **~3 GB**.

## Verify on your hardware

Estimates get you in the ballpark; the real number depends on your GPU, driver, and context length. To confirm, run your agent and watch the process:

- **Windows:** Task Manager → *Details* tab → add the **GPU Memory (dedicated)** and **Memory (working set)** columns for `tryll_server.exe`. Or use Performance Monitor's `GPU Process Memory \ Dedicated Usage`.
- Drive a few realistic, long conversations — the **peak** during the longest dialog is what you must fit, since the KV cache is largest then.

## Common pitfalls

- **Forgetting the KV cache.** Weights are only part of the VRAM bill; a long conversation can add ~1–2 GB on top. Size for your *longest* expected dialog.
- **Assuming voice needs a bigger GPU.** STT/TTS use **no VRAM** — they only need RAM. A bigger TTS voice costs RAM, not GPU.
- **Pinned models never freeing.** A [pinned](https://docs.tryllengine.com/how-to/pin-and-unpin-models.html) model stays resident for the whole server lifetime. If you pin several, their footprints add up — budget for all pinned models at once, not one at a time.
- **CPU-only builds.** Without GPU offload, weights *and* KV cache live in **RAM** instead of VRAM — shift those numbers from the VRAM column to the RAM column.
- **Multiple `Generate` nodes / agents.** Each model context carries its own KV cache; two agents on the same model still pay two KV caches.

## Related

- [Concept: Models and Inference Engines](https://docs.tryllengine.com/concepts/models-and-inference-engines.html)
- [Concept: Projection and Token Budgets](https://docs.tryllengine.com/concepts/projection-and-token-budgets.html)
- [How-to: Pin and Unpin Models](https://docs.tryllengine.com/how-to/pin-and-unpin-models.html)
- [How-to: Use Your Own Local Model](https://docs.tryllengine.com/how-to/use-your-own-local-model.html)
- [Reference: Model Management](https://docs.tryllengine.com/reference/model-management.html)
