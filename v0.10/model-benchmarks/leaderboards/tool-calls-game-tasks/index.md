[All model benchmarks](https://docs.tryllengine.com/v0.10/model-benchmarks/index.html)

# Tool calls, game tasks

Tryll game suites, one greedy pass

Single player lines from game tasks: spoken commands, opening doors, querying a game log, NPC reactions, haggling and reading a price. A line counts only when the calls match an accepted answer exactly: the right tool, every required argument, the right values and types, nothing extra. Lines where no tool fits count when the model makes no call. One pass per line at temperature 0.

*Every model on this page was measured on NVIDIA GeForce RTX 4080 · Vulkan.*

## Tool calls, game tasks

## Measured models

| Model                                 | Right | Interval       | Rank range   | Cases      | Most common mistake                 | Family    |
| ------------------------------------- | ----- | -------------- | ------------ | ---------- | ----------------------------------- | --------- |
| Ministral 3 3B Instruct (Q4_K_M)      | 91.7% | 87.1% to 96.2% | #1–4 of 12   | 121 of 132 | missed a call (6)                   | Ministral |
| Ministral 3 8B Instruct (Q4_K_M)      | 91.7% | 86.4% to 96.2% | #1–4 of 12   | 121 of 132 | missed a call (10)                  | Ministral |
| Bonsai 8B (Q2_0 2-bit)                | 87.1% | 81.1% to 92.4% | #1–6 of 12   | 115 of 132 | missed a call (16)                  | Bonsai    |
| Qwen 3.5 4B (Q4_K_M)                  | 86.4% | 80.3% to 91.7% | #1–6 of 12   | 114 of 132 | missed a call (10)                  | Qwen      |
| Gemma 4 E4B Instruct (Q4_K_M)         | 81.1% | 74.2% to 87.1% | #3–8 of 12   | 107 of 132 | missed a call (23)                  | Gemma     |
| Llama 3.1 8B Instruct (Q4_K_M)        | 78.0% | 70.5% to 84.8% | #3–10 of 12  | 103 of 132 | made a call it should not have (18) | Llama     |
| Ling 3.0 Tiny (Q4_K_M)                | 76.5% | 69.7% to 84.1% | #5–10 of 12  | 101 of 132 | missed a call (15)                  | Ling      |
| Granite 4.2 8B Instruct (Q4_K_M)      | 75.8% | 68.2% to 82.6% | #5–10 of 12  | 100 of 132 | missed a call (28)                  | Granite   |
| Bonsai 8B (Q1_0 1-bit)                | 72.7% | 65.2% to 80.3% | #6–10 of 12  | 96 of 132  | missed a call (32)                  | Bonsai    |
| Granite 4.2 3B Instruct (Q4_K_M)      | 69.7% | 61.4% to 77.3% | #6–10 of 12  | 92 of 132  | missed a call (35)                  | Granite   |
| Gemma 4 E2B Instruct QAT (UD-Q4_K_XL) | 59.1% | 50.8% to 67.4% | #11–12 of 12 | 78 of 132  | missed a call (50)                  | Gemma     |
| Llama 3.2 3B Instruct (Q4_K_M)        | 47.7% | 39.4% to 56.1% | #11–12 of 12 | 63 of 132  | made a call it should not have (22) | Llama     |

## Model cards

- [Ministral 3 3B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.10/model-benchmarks/models/ministral_3_3b/index.html)
- [Ministral 3 8B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.10/model-benchmarks/models/ministral_3_8b/index.html)
- [Bonsai 8B (Q2_0 2-bit)](https://docs.tryllengine.com/v0.10/model-benchmarks/models/bonsai_8b_q2_0/index.html)
- [Qwen 3.5 4B (Q4_K_M)](https://docs.tryllengine.com/v0.10/model-benchmarks/models/qwen_3.5_4b/index.html)
- [Gemma 4 E4B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.10/model-benchmarks/models/gemma_4_e4b/index.html)
- [Llama 3.1 8B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.10/model-benchmarks/models/llama_3.1_8b/index.html)
- [Ling 3.0 Tiny (Q4_K_M)](https://docs.tryllengine.com/v0.10/model-benchmarks/models/ling_3.0_tiny/index.html)
- [Granite 4.2 8B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.10/model-benchmarks/models/granite_4.2_8b/index.html)
- [Bonsai 8B (Q1_0 1-bit)](https://docs.tryllengine.com/v0.10/model-benchmarks/models/bonsai_8b_q1_0/index.html)
- [Granite 4.2 3B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.10/model-benchmarks/models/granite_4.2_3b/index.html)
- [Gemma 4 E2B Instruct QAT (UD-Q4_K_XL)](https://docs.tryllengine.com/v0.10/model-benchmarks/models/gemma_4_e2b_qat/index.html)
- [Llama 3.2 3B Instruct (Q4_K_M)](https://docs.tryllengine.com/v0.10/model-benchmarks/models/llama_3.2_3b/index.html)

*No tool support: SmolLM2 1.7B Instruct (Q4_K_M).*
