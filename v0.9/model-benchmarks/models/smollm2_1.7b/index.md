[All model benchmarks](https://docs.tryllengine.com/v0.9/model-benchmarks/index.html)

# SmolLM2 1.7B Instruct (Q4_K_M)

Model card

**As an immersion guard, blocks 36 in 100 general attacks and 28 in 100 world attacks at a 5% false-block budget. Writes 354 tokens per second and needs 2.0 GB of VRAM at 8K context.**

SmolLM2 · 1.7B · Q4_K_M · 1.1 GB file · HuggingFaceTB/SmolLM2-1.7B-Instruct-GGUF

Measured 2026-10-08 · Tryll 0.8.0-DEV · llama.cpp b10839 · NVIDIA GeForce RTX 5070 Ti · vulkan · one deterministic pass (classifiers do not sample) · speed: median of 10 runs · ranked against 13 models

## Plain words

*What the measured numbers mean for a game. Intervals and ranks are under In the field.*

**36 in 100** · Classification · General guard. Blocks 36 in 100 attempts to pull the NPC out of the game: asking whether it is an AI, jailbreaks, prompt leaks, coding requests and format tricks. *Normal player lines wrongly blocked: 4 in 100 on held-out lines (the budget is 5 in 100, set on a separate tuning set).*

**28 in 100** · Classification · World guard. Blocks 28 in 100 lines that do not belong in the setting, such as paying by phone in a medieval town or asking how long chicken keeps in the fridge. *Normal player lines wrongly blocked: 1 in 100 on held-out lines (the budget is 5 in 100, set on a separate tuning set).*

*World guard is measured on one setting, a medieval merchant. A different world needs its own check.*

**13,668 tokens/s** · Performance · Prefill. Reads a 4,058-token prompt in 0.30 s. The first token reaches the player after 0.32 s. *A 1,193-token prompt: first token after 0.10 s.*

**354 tokens/s** · Performance · Generation. Writes 354 tokens per second after a 1K-token prompt, and 287 after a 4K-token prompt. *200 tokens take 0.6 s.*

**2.0 GB VRAM** · Performance · Memory. Weights 1.05 GB, KV cache 0.86 GB for 8,192 tokens (q8_0), compute buffers 0.07 GB. *Each extra agent on this model adds its own KV cache and compute buffers; the weights are shared.*

*Speed and memory were measured on one GPU, named in the header. Other hardware gives other numbers.*

### Not measured yet

- **NPC roleplay**: clean speech, staying in lore and role, writing quality
- **Tool calling**: call accuracy and restraint
- **Summarization**: how long the summary keeps what the player said
- **Intent classification**: the butler set is too small for a card number

## In the field

*Where this model sits among the measured models. Right is better. The blue bar is the 95% interval. Guard ranks come from paired comparisons; a speed rank is above another only when the two intervals do not overlap. Close models share a range.*

**General guard** · #7–12 of 13 · *setting-invariant classifier*

**World guard** · #13 of 13 · *measured on 1 setting*

**Prefill, 1K prompt** · #1 of 13 · *measured on one GPU*

**Prefill, 4K prompt** · #1 of 13 · *measured on one GPU*

**Generation, 1K prompt** · #1 of 13 · *measured on one GPU*

**Generation, 4K prompt** · #1 of 13 · *measured on one GPU*

**VRAM at 8K context** · #2 of 13 · *one agent, 8K context, q8_0 KV cache*

### Leaderboards

- [General guard](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/general-guard/index.html)
- [World guard](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/world-guard/index.html)
- [Prefill, 1K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/prefill-1k-prompt/index.html)
- [Prefill, 4K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/prefill-4k-prompt/index.html)
- [Generation, 1K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/generation-1k-prompt/index.html)
- [Generation, 4K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/generation-4k-prompt/index.html)
- [VRAM at 8K context](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/vram-at-8k-context/index.html)
