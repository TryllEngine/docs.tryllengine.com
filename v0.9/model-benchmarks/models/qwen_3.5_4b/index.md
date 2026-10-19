[All model benchmarks](https://docs.tryllengine.com/v0.9/model-benchmarks/index.html)

# Qwen 3.5 4B (Q4_K_M)

Model card

**As an immersion guard, blocks 79 in 100 general attacks and 90 in 100 world attacks at a 5% false-block budget. Writes 174 tokens per second and needs 3.0 GB of VRAM at 8K context.**

Qwen · 4B · Q4_K_M · 2.7 GB file · unsloth/Qwen3.5-4B-GGUF

Measured 2026-10-08 · Tryll 0.8.0-DEV · llama.cpp b10839 · NVIDIA GeForce RTX 5070 Ti · vulkan · one deterministic pass (classifiers do not sample) · speed: median of 10 runs · ranked against 13 models

## Plain words

*What the measured numbers mean for a game. Intervals and ranks are under In the field.*

**79 in 100** · Classification · General guard. Blocks 79 in 100 attempts to pull the NPC out of the game: asking whether it is an AI, jailbreaks, prompt leaks, coding requests and format tricks. *Normal player lines wrongly blocked: 5 in 100 on held-out lines (the budget is 5 in 100, set on a separate tuning set).*

**90 in 100** · Classification · World guard. Blocks 90 in 100 lines that do not belong in the setting, such as paying by phone in a medieval town or asking how long chicken keeps in the fridge. *Normal player lines wrongly blocked: 5 in 100 on held-out lines (the budget is 5 in 100, set on a separate tuning set).*

*World guard is measured on one setting, a medieval merchant. A different world needs its own check.*

**5,398 tokens/s** · Performance · Prefill. Reads a 3,981-token prompt in 0.74 s. The first token reaches the player after 0.77 s. *A 1,156-token prompt: first token after 0.26 s.*

**174 tokens/s** · Performance · Generation. Writes 174 tokens per second after a 1K-token prompt, and 172 after a 4K-token prompt. *200 tokens take 1.2 s.*

**3.0 GB VRAM** · Performance · Memory. Weights 2.73 GB, KV cache 0.20 GB for 8,192 tokens (q8_0), compute buffers 0.08 GB. *Each extra agent on this model adds its own KV cache and compute buffers; the weights are shared.*

*Speed and memory were measured on one GPU, named in the header. Other hardware gives other numbers.*

### Not measured yet

- **NPC roleplay**: clean speech, staying in lore and role, writing quality
- **Tool calling**: call accuracy and restraint
- **Summarization**: how long the summary keeps what the player said
- **Intent classification**: the butler set is too small for a card number

## In the field

*Where this model sits among the measured models. Right is better. The blue bar is the 95% interval. Guard ranks come from paired comparisons; a speed rank is above another only when the two intervals do not overlap. Close models share a range.*

**General guard** · #2–4 of 13 · *setting-invariant classifier*

**World guard** · #1–4 of 13 · *measured on 1 setting*

**Prefill, 1K prompt** · #7–9 of 13 · *measured on one GPU*

**Prefill, 4K prompt** · #7 of 13 · *measured on one GPU*

**Generation, 1K prompt** · #9 of 13 · *measured on one GPU*

**Generation, 4K prompt** · #9 of 13 · *measured on one GPU*

**VRAM at 8K context** · #7 of 13 · *one agent, 8K context, q8_0 KV cache*

### Leaderboards

- [General guard](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/general-guard/index.html)
- [World guard](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/world-guard/index.html)
- [Prefill, 1K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/prefill-1k-prompt/index.html)
- [Prefill, 4K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/prefill-4k-prompt/index.html)
- [Generation, 1K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/generation-1k-prompt/index.html)
- [Generation, 4K prompt](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/generation-4k-prompt/index.html)
- [VRAM at 8K context](https://docs.tryllengine.com/v0.9/model-benchmarks/leaderboards/vram-at-8k-context/index.html)
