[All model benchmarks](https://docs.tryllengine.com/v0.9/model-benchmarks/index.html)

# General guard

setting-invariant classifier

The share of attempts to pull the NPC out of the game that the guard blocks: asking whether it is an AI, jailbreaks, prompt leaks, coding requests and format tricks. Each model's threshold is set so that at most 5 in 100 normal player lines are blocked on a tuning set; the numbers here are on separate, held-out lines.

## General guard

## Measured models

| Model                                 | Block rate | Interval       | Rank range   | False blocks | Family    |
| ------------------------------------- | ---------- | -------------- | ------------ | ------------ | --------- |
| Gemma 4 E4B Instruct (Q4_K_M)         | 90.8%      | 86.6% to 94.7% | #1 of 13     | 8.5%         | Gemma     |
| Qwen 3.5 4B (Q4_K_M)                  | 78.6%      | 72.6% to 84.1% | #2–4 of 13   | 4.5%         | Qwen      |
| Llama 3.1 8B Instruct (Q4_K_M)        | 77.0%      | 70.7% to 82.6% | #2–4 of 13   | 10.7%        | Llama     |
| Gemma 4 E2B Instruct QAT (UD-Q4_K_XL) | 75.0%      | 68.6% to 80.8% | #2–4 of 13   | 8.5%         | Gemma     |
| Bonsai 8B (Q2_0 2-bit)                | 62.8%      | 55.7% to 69.6% | #5–6 of 13   | 2.3%         | Bonsai    |
| Ministral 3 8B Instruct (Q4_K_M)      | 59.7%      | 52.2% to 66.8% | #5–6 of 13   | 4.0%         | Ministral |
| Ministral 3 3B Instruct (Q4_K_M)      | 41.8%      | 35.1% to 49.2% | #7–10 of 13  | 8.5%         | Ministral |
| SmolLM2 1.7B Instruct (Q4_K_M)        | 35.7%      | 29.2% to 42.2% | #7–12 of 13  | 4.0%         | SmolLM2   |
| Ling 3.0 Tiny (Q4_K_M)                | 34.2%      | 27.4% to 40.9% | #7–11 of 13  | 3.4%         | Ling      |
| Llama 3.2 3B Instruct (Q4_K_M)        | 32.1%      | 25.4% to 38.8% | #7–12 of 13  | 1.1%         | Llama     |
| Bonsai 8B (Q1_0 1-bit)                | 31.6%      | 25.0% to 38.4% | #8–12 of 13  | 4.5%         | Bonsai    |
| Granite 4.2 3B Instruct (Q4_K_M)      | 26.5%      | 20.4% to 33.3% | #9–13 of 13  | 0.6%         | Granite   |
| Granite 4.2 8B Instruct (Q4_K_M)      | 21.9%      | 16.4% to 28.4% | #12–13 of 13 | 0.6%         | Granite   |

## Model cards

- [Gemma 4 E4B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/gemma_4_e4b/index.html)
- [Qwen 3.5 4B (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/qwen_3.5_4b/index.html)
- [Llama 3.1 8B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/llama_3.1_8b/index.html)
- [Gemma 4 E2B Instruct QAT (UD-Q4_K_XL)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/gemma_4_e2b_qat/index.html)
- [Bonsai 8B (Q2_0 2-bit)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/bonsai_8b_q2_0/index.html)
- [Ministral 3 8B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/ministral_3_8b/index.html)
- [Ministral 3 3B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/ministral_3_3b/index.html)
- [SmolLM2 1.7B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/smollm2_1.7b/index.html)
- [Ling 3.0 Tiny (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/ling_3.0_tiny/index.html)
- [Llama 3.2 3B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/llama_3.2_3b/index.html)
- [Bonsai 8B (Q1_0 1-bit)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/bonsai_8b_q1_0/index.html)
- [Granite 4.2 3B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/granite_4.2_3b/index.html)
- [Granite 4.2 8B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.9/model-benchmarks/models/granite_4.2_8b/index.html)
