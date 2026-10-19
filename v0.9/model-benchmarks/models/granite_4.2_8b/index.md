[All model benchmarks](https://docs.tryllengine.com/v0.9/model-benchmarks/index.html)

# Granite 4.2 8B Instruct (Q4_K_M)

Model card

**As an immersion guard, blocks 22 in 100 general attacks and 88 in 100 world attacks at a 5% false-block budget. Writes 112 tokens per second and needs 6.4 GB of VRAM at 8K context.**

Granite · 8B · Q4_K_M · 5.5 GB file · bartowski/granite-4.2-8b-GGUF

Measured 2026-10-08 · Tryll 0.8.0-DEV · llama.cpp b10839 · NVIDIA GeForce RTX 5070 Ti · vulkan · one deterministic pass (classifiers do not sample) · speed: median of 10 runs · ranked against 13 models

## Plain words

*What the measured numbers mean for a game. Intervals and ranks are under In the field.*

**22 in 100** · Classification · General guard. Blocks 22 in 100 attempts to pull the NPC out of the game: asking whether it is an AI, jailbreaks, prompt leaks, coding requests and format tricks. *Normal player lines wrongly blocked: 1 in 100 on held-out lines (the budget is 5 in 100, set on a separate tuning set).*

**88 in 100** · Classification · World guard. Blocks 88 in 100 lines that do not belong in the setting, such as paying by phone in a medieval town or asking how long chicken keeps in the fridge. *Normal player lines wrongly blocked: 3 in 100 on held-out lines (the budget is 5 in 100, set on a separate tuning set).*

*World guard is measured on one setting, a medieval merchant. A different world needs its own check.*

**4,130 tokens/s** · Performance · Prefill. Reads a 4,018-token prompt in 0.97 s. The first token reaches the player after 1.00 s. *A 1,169-token prompt: first token after 0.30 s.*

**112 tokens/s** · Performance · Generation. Writes 112 tokens per second after a 1K-token prompt, and 106 after a 4K-token prompt. *200 tokens take 1.8 s.*

**6.4 GB VRAM** · Performance · Memory. Weights 5.54 GB, KV cache 0.71 GB for 8,192 tokens (q8_0), compute buffers 0.10 GB. *Each extra agent on this model adds its own KV cache and compute buffers; the weights are shared.*

*Speed and memory were measured on one GPU, named in the header. Other hardware gives other numbers.*

### Not measured yet

- **NPC roleplay**: clean speech, staying in lore and role, writing quality
- **Tool calling**: call accuracy and restraint
- **Summarization**: how long the summary keeps what the player said
- **Intent classification**: the butler set is too small for a card number

## In the field

*Where this model sits among the measured models. Right is better. The blue bar is the 95% interval. Guard ranks come from paired comparisons; a speed rank is above another only when the two intervals do not overlap. Close models share a range.*

**General guard** · #12–13 of 13 · *setting-invariant classifier*

**World guard** · #1–6 of 13 · *measured on 1 setting*

**Prefill, 1K prompt** · #13 of 13 · *measured on one GPU*

**Prefill, 4K prompt** · #13 of 13 · *measured on one GPU*

**Generation, 1K prompt** · #12 of 13 · *measured on one GPU*

**Generation, 4K prompt** · #12 of 13 · *measured on one GPU*

**VRAM at 8K context** · #13 of 13 · *one agent, 8K context, q8_0 KV cache*

### Leaderboards

- [General guard](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/general-guard/index.html)
- [World guard](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/world-guard/index.html)
- [Prefill, 1K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/prefill-1k-prompt/index.html)
- [Prefill, 4K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/prefill-4k-prompt/index.html)
- [Generation, 1K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/generation-1k-prompt/index.html)
- [Generation, 4K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/generation-4k-prompt/index.html)
- [VRAM at 8K context](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/vram-at-8k-context/index.html)
