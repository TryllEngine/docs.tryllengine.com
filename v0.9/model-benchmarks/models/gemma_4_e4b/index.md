[All model benchmarks](https://docs.tryllengine.com/v0.9/model-benchmarks/index.html)

# Gemma 4 E4B Instruct (Q4_K_M)

Model card

**As an immersion guard, blocks 91 in 100 general attacks and 90 in 100 world attacks at a 5% false-block budget. Writes 143 tokens per second and needs 5.8 GB of VRAM at 8K context.**

Gemma · E4B · Q4_K_M · 5.4 GB file · bartowski/google_gemma-4-E4B-it-GGUF

Measured 2026-10-07 · Tryll 0.8.0-DEV · llama.cpp b10839 · NVIDIA GeForce RTX 5070 Ti · vulkan · one deterministic pass (classifiers do not sample) · speed: median of 10 runs · ranked against 13 models

## Plain words

*What the measured numbers mean for a game. Intervals and ranks are under In the field.*

**91 in 100** · Classification · General guard. Blocks 91 in 100 attempts to pull the NPC out of the game: asking whether it is an AI, jailbreaks, prompt leaks, coding requests and format tricks. *Normal player lines wrongly blocked: 8 in 100 on held-out lines (the budget is 5 in 100, set on a separate tuning set).*

**90 in 100** · Classification · World guard. Blocks 90 in 100 lines that do not belong in the setting, such as paying by phone in a medieval town or asking how long chicken keeps in the fridge. *Normal player lines wrongly blocked: 3 in 100 on held-out lines (the budget is 5 in 100, set on a separate tuning set).*

*World guard is measured on one setting, a medieval merchant. A different world needs its own check.*

**4,904 tokens/s** · Performance · Prefill. Reads a 3,889-token prompt in 0.79 s. The first token reaches the player after 0.82 s. *A 1,124-token prompt: first token after 0.27 s.*

**143 tokens/s** · Performance · Generation. Writes 143 tokens per second after a 1K-token prompt, and 139 after a 4K-token prompt. *200 tokens take 1.4 s.*

**5.8 GB VRAM** · Performance · Memory. Weights 5.39 GB, KV cache 0.25 GB for 8,192 tokens (q8_0), compute buffers 0.14 GB. *Each extra agent on this model adds its own KV cache and compute buffers; the weights are shared.*

*Speed and memory were measured on one GPU, named in the header. Other hardware gives other numbers.*

### Not measured yet

- **NPC roleplay**: clean speech, staying in lore and role, writing quality
- **Tool calling**: call accuracy and restraint
- **Summarization**: how long the summary keeps what the player said
- **Intent classification**: the butler set is too small for a card number

## In the field

*Where this model sits among the measured models. Right is better. The blue bar is the 95% interval. Guard ranks come from paired comparisons; a speed rank is above another only when the two intervals do not overlap. Close models share a range.*

**General guard** · #1 of 13 · *setting-invariant classifier*

**World guard** · #1–6 of 13 · *measured on 1 setting*

**Prefill, 1K prompt** · #8–11 of 13 · *measured on one GPU*

**Prefill, 4K prompt** · #8 of 13 · *measured on one GPU*

**Generation, 1K prompt** · #10 of 13 · *measured on one GPU*

**Generation, 4K prompt** · #10 of 13 · *measured on one GPU*

**VRAM at 8K context** · #11 of 13 · *one agent, 8K context, q8_0 KV cache*

### Leaderboards

- [General guard](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/general-guard/index.html)
- [World guard](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/world-guard/index.html)
- [Prefill, 1K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/prefill-1k-prompt/index.html)
- [Prefill, 4K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/prefill-4k-prompt/index.html)
- [Generation, 1K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/generation-1k-prompt/index.html)
- [Generation, 4K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/generation-4k-prompt/index.html)
- [VRAM at 8K context](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/vram-at-8k-context/index.html)
