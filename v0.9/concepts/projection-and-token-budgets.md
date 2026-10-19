# Projection and Token Budgets

A Tryll [agent](https://docs.tryllengine.com/v0.9/reference/glossary.html#agent) keeps a growing history of every turn — the user's messages, the assistant's replies, retrieved knowledge, tool calls the graph recorded. The model, on the other hand, wants a fixed, bounded list of chat turns. The thing that translates one into the other is **projection**. Getting projection right is how Tryll keeps conversations coherent turn after turn without burning its context window or its KV cache.

## The flow: dialog → prompt

```
flowchart LR
    D[Dialog<br>growing list of turns] --> PR[Projection<br>per node]
    PR --> P["Prompt<br>ordered chat messages"]
    P --> M[Model<br>incremental KV update]
    M --> Gen[Generate]
```

Each node in a graph has its own projection. On every turn, before the node calls the model, projection takes the dialog and produces a prompt: an ordered list of chat messages (role + text). The model then compares that prompt against what is cached in its KV state and re-processes only what changed.

Two consecutive turns usually share 99% of their prompt, so projection is designed to leave the shared prefix alone and only re-tokenise what is new. This is what makes generation feel fast.

## Two projection variants

Tryll ships two:

| Variant       | Produces                                                                                                                                                                                                                                                                                                                                                          | Used by           |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- |
| **Default**   | Dialog → plain chat sequence. Any instruction-kind [slots](https://docs.tryllengine.com/v0.9/concepts/slots-and-inter-node-value-passing.html) and `KnowledgeComponent`s from the *last* turn are rendered via the node's [Mustache template](https://docs.tryllengine.com/v0.9/how-to/use-mustache-templates.html) and spliced in at the configured `placement`. | `Generate` nodes. |
| **Tool-call** | Same as default, but bakes the tool schema into the model's chat template for the current turn, and projects prior tool-call turns as complete native pairs per `disposition` (`ToolCallDisposition`; see [Tool Calling](https://docs.tryllengine.com/v0.9/concepts/tool-calling.html#tool-call-history-complete-pairs-never-dangling)).                          | `ToolCall` nodes. |

Both then apply a token budget with hysteresis — the interesting trimming.

## Why projection is per-node, not per-agent

Two nodes in the same graph that use the same model still see *different* prompts. A `Generate` node sees the full dialog projected plainly; a `ToolCall` node sees the same dialog with a bolted-on tool schema for the current turn. If both nodes shared the same KV state, they would invalidate each other's cache on every turn.

So each node keeps its own KV state for the model it uses. You pay a little more memory; in return, each node's cache stays warm across turns.

## Token budgeting with hysteresis

This is the part that keeps a long conversation inside the context window.

Every model has a **context size** — the total number of tokens it can attend to in one forward pass. Tryll reserves a slice at the end for the generation itself (`generation_reserve`). The remaining space defines the **high water mark**:

```
high_water = context_size − generation_reserve
```

Tryll watches the total token count of the projected prompt. When it exceeds `high_water`, it starts trimming the oldest *non-system* messages — but not down to `high_water`. It trims down to a **low water mark** (about 75 % of high):

```
flowchart LR
    In["total_tokens"] -->|over high_water| Trim["drop oldest<br>non-system turns"]
    Trim -->|"until ≤ low_water"| Out[clean prompt]
    In -->|"≤ high_water"| Out
```

Why two marks? Because trimming to the threshold means the *next* turn almost certainly crosses it again, so you pay a fresh KV-cache rebuild every turn. Trimming to 75 % buys you roughly 25 % of the context window worth of new conversation before the next trim — amortising the cost. That is "hysteresis" in the plumbing sense: the system resists bouncing back and forth across the boundary.

A detail worth knowing: trimming is one-way. Once the oldest turns have been dropped from the model's view, they do not come back. The agent's dialog still has them; the model just stops seeing them.

## What "tokens" mean here

Not characters. Each model brings its own tokeniser — a 200-token message for Llama is not a 200-token message for Phi. Tokens are counted once per message and reused across turns, so the budget tracks the model's view accurately without re-tokenising every time.

There is also a small fixed **template overhead** added per message (currently 8 tokens) to account for chat-template glue (role headers, end-of-turn markers). The prompt the model actually sees is slightly longer than the sum of the visible text, and the budget accounts for this.

## Why re-decoding is cheap

Each turn, the node compares the new prompt against the tokens already in its KV cache, keeps the matching prefix, drops the stale tail, and re-runs the model over only the new suffix. On a normal chat turn, the only new thing is the user's latest line, so almost nothing is re-processed. That is why generation feels responsive even on a modest local GPU.

The ugly case is when you change something in the *middle* of the history — for example, if a guardrail re-orders messages, or a RAG retrieve changes what the Mustache template renders around the user message. In that case, more of the cache is dropped and more must be re-processed. Neither is fatal, but it is worth knowing when you design a graph that rewrites past turns.

## Edges and pitfalls

- **The system message never gets trimmed.** It stays pinned at the head. Writing a 3000-token system prompt will permanently shrink your usable dialog window — keep system text lean.
- **Knowledge is projected from the *last* interaction only.** A `Retrieve` node that ran three turns ago does not contribute to this turn's prompt. If you need older knowledge, re-retrieve it or manually surface it.
- **Tool-call history is always a complete pair (or omitted).** Under `Acknowledge` / `NotifyAndAcknowledge` / `PauseAndAcknowledge` / `AwaitResult`, a past tool turn projects as `assistant(tool_calls)` + `tool(result)` (plus a historical empty assistant closer for templates like Mistral). `RouteOnly` / `Notify` / `Pause` drop the tool syntax but still close historical turns. Token-budget trim treats the call+result block as an atomic unit — see [Tool Calling](https://docs.tryllengine.com/v0.9/concepts/tool-calling.html#tool-call-history-complete-pairs-never-dangling).
- **You can't change a node's model mid-agent.** Each node's KV cache is bound to the model it was created with. Create a new agent if you need a different model.

## Rolling summaries

Once the oldest turns have been dropped from the prompt, the model stops seeing them. The dialog still stores them. A [`Summarize`](https://docs.tryllengine.com/v0.9/reference/nodes/summarize.html) node folds those ageing interactions into a rolling summary while they are still inside the window, which is why `trigger_tokens` has to sit below the trim threshold.

On later turns, every Generate-class node appends the newest summary to its system message, under a label that treats the text as untrusted dialogue data, and replays history only from the fold boundary. The system message stays pinned, so the summary stays in the prompt while the verbatim turns it replaced can leave.

Place the node last, after the reply the player is waiting on, and size `trigger_tokens` below the consuming node's trim threshold. The recipe is [Keep long conversations coherent](https://docs.tryllengine.com/v0.9/how-to/keep-long-conversations-coherent.html).

A summary carries the thread of the scene: where it is, what was agreed, what is still open. Exact recall of one fact much later is a retrieval job — see [RAG](https://docs.tryllengine.com/v0.9/concepts/rag.html).

## Related

- [How-to: Keep long conversations coherent](https://docs.tryllengine.com/v0.9/how-to/keep-long-conversations-coherent.html)
- [Summarize node](https://docs.tryllengine.com/v0.9/reference/nodes/summarize.html)
- [Reference: Agent Parameters](https://docs.tryllengine.com/v0.9/reference/agent-parameters.html)
- [Reference: Model Management](https://docs.tryllengine.com/v0.9/reference/model-management.html)
- [Concept: Models and Inference Engines](https://docs.tryllengine.com/v0.9/concepts/models-and-inference-engines.html)
- [Concept: Workflows, Graphs, and Nodes](https://docs.tryllengine.com/v0.9/concepts/workflows-graphs-nodes.html)
- [Concept: RAG](https://docs.tryllengine.com/v0.9/concepts/rag.html)
