# Model Benchmarks

Provisional · immersion guard, speed and memory

How catalog models do on the jobs Tryll gives them, measured by the Tryll team on our own test sets. Measured so far: the immersion guard, General guard (attempts to pull the NPC out of the game) and World guard (lines that do not belong in the setting), and speed and memory: how fast a model reads its prompt and writes its answer, and how much graphics memory it needs. NPC roleplay, tool calling and summarization will follow.

*The numbers are provisional: 13 models. The guard is measured once per model (it reads probabilities and does not sample), at a 5 in 100 false-block budget, with World guard on one game setting. Speed and memory are measured on NVIDIA GeForce RTX 5070 Ti with Vulkan; other hardware gives other numbers.*

## Model cards

- [Bonsai 8B (Q1_0 1-bit)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/bonsai_8b_q1_0/index.html) · general guard 31.6% · world guard 60.5% · generation 238 tok/s · VRAM 1.90 GB
- [Bonsai 8B (Q2_0 2-bit)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/bonsai_8b_q2_0/index.html) · general guard 62.8% · world guard 86.4% · generation 221 tok/s · VRAM 3.05 GB
- [Gemma 4 E2B Instruct QAT (UD-Q4_K_XL)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/gemma_4_e2b_qat/index.html) · general guard 75.0% · world guard 76.5% · generation 259 tok/s · VRAM 2.81 GB
- [Gemma 4 E4B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/gemma_4_e4b/index.html) · general guard 90.8% · world guard 90.1% · generation 143 tok/s · VRAM 5.78 GB
- [Granite 4.2 3B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/granite_4.2_3b/index.html) · general guard 26.5% · world guard 72.8% · generation 200 tok/s · VRAM 2.74 GB
- [Granite 4.2 8B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/granite_4.2_8b/index.html) · general guard 21.9% · world guard 87.7% · generation 112 tok/s · VRAM 6.35 GB
- [Ling 3.0 Tiny (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/ling_3.0_tiny/index.html) · general guard 34.2% · world guard 61.7% · generation 267 tok/s · VRAM 5.03 GB
- [Llama 3.1 8B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/llama_3.1_8b/index.html) · general guard 77.0% · world guard 80.2% · generation 134 tok/s · VRAM 5.60 GB
- [Llama 3.2 3B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/llama_3.2_3b/index.html) · general guard 32.1% · world guard 64.2% · generation 256 tok/s · VRAM 2.58 GB
- [Ministral 3 3B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/ministral_3_3b/index.html) · general guard 41.8% · world guard 71.6% · generation 189 tok/s · VRAM 2.69 GB
- [Ministral 3 8B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/ministral_3_8b/index.html) · general guard 59.7% · world guard 82.7% · generation 105 tok/s · VRAM 5.91 GB
- [Qwen 3.5 4B (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/qwen_3.5_4b/index.html) · general guard 78.6% · world guard 90.1% · generation 174 tok/s · VRAM 3.01 GB
- [SmolLM2 1.7B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/smollm2_1.7b/index.html) · general guard 35.7% · world guard 28.4% · generation 354 tok/s · VRAM 1.98 GB

## Leaderboards

- [General guard](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/general-guard/index.html) · setting-invariant classifier
- [World guard](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/world-guard/index.html) · measured on 1 setting
- [Prefill, 1K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/prefill-1k-prompt/index.html) · measured on one GPU
- [Prefill, 4K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/prefill-4k-prompt/index.html) · measured on one GPU
- [Generation, 1K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/generation-1k-prompt/index.html) · measured on one GPU
- [Generation, 4K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/generation-4k-prompt/index.html) · measured on one GPU
- [VRAM at 8K context](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/vram-at-8k-context/index.html) · one agent, 8K context, q8_0 KV cache
