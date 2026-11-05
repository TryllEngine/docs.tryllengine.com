# Dialog History and the Context Window

A Tryll [agent](https://docs.tryllengine.com/v0.10/reference/glossary.html#agent) remembers the whole conversation: every line the player said, every reply, every tool call. A model can only read a limited number of tokens at once, its **context window**. Before every model call, each node turns the full history into a **prompt** that fits its window.

This page explains how that prompt is put together, what gets cut when a conversation outgrows the window, and why Tryll cuts more than it has to. Knowing this lets you size **Context Size** and **Max Tokens**, write system prompts that leave room for the conversation, and read what the [Turn Inspector](https://docs.tryllengine.com/v0.10/reference/turn-inspector.html) shows you.

## Dialog history and the prompt are different things

|                   | Dialog history                                                                                                                                                                                                   | Prompt                                                                                                                                |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| **Belongs to**    | The agent                                                                                                                                                                                                        | One node, for one turn                                                                                                                |
| **Holds**         | Every turn since the agent was created (or since you last seeded it)                                                                                                                                             | The system message, the turns that still fit, and the current turn                                                                    |
| **Grows**         | Without limit                                                                                                                                                                                                    | Never past the node's token budget                                                                                                    |
| **You see it in** | [Export and import dialog](https://docs.tryllengine.com/v0.10/how-to/export-and-import-dialog.html), [Seed and edit dialog history](https://docs.tryllengine.com/v0.10/how-to/seed-and-edit-dialog-history.html) | The **Prompt & output** block of the [Turn Inspector](https://docs.tryllengine.com/v0.10/reference/turn-inspector.html#prompt-output) |

When a long conversation no longer fits, turns leave the **prompt**. They stay in the **dialog history**: an export still contains them, and a [rolling summary](#rolling-summaries-keep-the-thread) can still fold them in.

```
flowchart LR
    H["Dialog history<br>every turn so far"] --> B["Build the prompt<br>(per node)"]
    B --> F["Fit the token budget<br>drop oldest turns if needed"]
    F --> P["Prompt<br>ordered chat messages"]
    P --> M["Model<br>reuses its KV cache,<br>reads only what changed"]
```

## How a node builds its prompt

Every node that calls a language model (Generate, Generate and Speak, Tool Call) builds its own prompt on every turn, in this order:

1. **The system message.** The node's **System Prompt**. If a [Summarize](https://docs.tryllengine.com/v0.10/reference/nodes/summarize.html) node has written a summary, it is appended here.
1. **Earlier turns.** For each earlier turn: the player's line exactly as they said it, then the agent's replies. A reply comes back only if the node that wrote it has **History Role** set to keep it (the default, `Assistant`). With `None`, later turns never see that output, which is what you want for a [hidden reasoning step](https://docs.tryllengine.com/v0.10/how-to/add-a-hidden-reasoning-step.html). A tool call comes back as a complete call-and-result pair, or not at all, depending on its **Disposition**. See [Tool-call history](https://docs.tryllengine.com/v0.10/concepts/tool-calling.html#tool-call-history-complete-pairs-never-dangling).
1. **The current turn.** The player's line (or the slot named by the node's **Input**). If the node has a **Template**, Tryll renders it with this turn's instructions, retrieved knowledge, [slots](https://docs.tryllengine.com/v0.10/concepts/slots-and-inter-node-value-passing.html) and [agent variables](https://docs.tryllengine.com/v0.10/concepts/agent-variables.html), and inserts the result before or after the player's line, as the node's **Placement** says. See [Use Mustache templates](https://docs.tryllengine.com/v0.10/how-to/use-mustache-templates.html).

A Tool Call node also adds its tool definitions for the current turn.

The rest of this page is about these three nodes. The LLM classifier nodes (Classify Intent (LLM), Classify Attributes (LLM)) and Summarize build short prompts of their own. Their reference pages describe them.

Two consequences are easy to miss:

- **Knowledge and instructions only frame the current turn.** When a turn becomes history, it comes back as the player's plain line. Knowledge a [Retrieve](https://docs.tryllengine.com/v0.10/reference/nodes/retrieve.html) node found three turns ago is not in this turn's prompt. If a node needs it, retrieve again, or keep the fact in an agent variable.
- **Each node sees its own version of the conversation.** Two nodes in one graph can have different system prompts, templates and history settings, so their prompts differ even when they use the same model.

## Each node keeps its own KV cache

Reading a prompt costs time. The model stores what it has already read in its **KV cache**. On the next turn it compares the new prompt with the cache, keeps the part that still matches from the start, and reads only the rest. On an ordinary turn only the end of the prompt is new, which is why replies start quickly even on a modest GPU.

Two nodes with different prompts would keep overwriting a shared cache, so each node has its own. That costs memory, which [Estimate the RAM and VRAM your setup needs](https://docs.tryllengine.com/v0.10/how-to/estimate-memory-footprint.html) and [Run many agents on a small GPU](https://docs.tryllengine.com/v0.10/how-to/run-many-agents-on-a-small-gpu.html) help you budget. In return, each node's cache stays warm from turn to turn. To free it while an agent is idle, see [Manage an agent's KV cache](https://docs.tryllengine.com/v0.10/how-to/manage-agent-kv-cache.html).

Anything that changes the **start** of the prompt makes the model read everything after it again. That happens when:

- old turns are trimmed (next section);
- a new rolling summary is written, because it changes the system message;
- you change the System Prompt at runtime with [Change agent parameters](https://docs.tryllengine.com/v0.10/how-to/change-agent-parameters.html);
- you edit or replace earlier history with [Seed and edit dialog history](https://docs.tryllengine.com/v0.10/how-to/seed-and-edit-dialog-history.html).

None of these is a problem now and then. A graph that does one on every turn pays for it on every turn.

## The token budget

A node's window is its **Context Size**. Leave it at `0` to use the model's own size. Part of the window is kept free for the answer the node is about to write, and **Max Tokens** sets that part. Everything else is the prompt budget:

```
prompt budget = Context Size − Max Tokens
```

For example, with Context Size 4096 and Max Tokens 512, the prompt can use 3,584 tokens. Max Tokens must be at least 1 and less than the Context Size. Both are fixed when the agent is created; to change them, create the agent again.

**Tokens are not characters.** Each model has its own tokenizer, so the same line costs a different number of tokens on Llama than on Gemma. Each message also costs a few extra tokens (Tryll counts 8) for the role headers and end-of-turn markers the model's chat template wraps around it. The budget counts both.

## Trimming, with hysteresis

When the prompt grows past the budget, the node drops the **oldest turns first**:

- A turn is dropped whole: the player's line, the replies, and any tool call with its result go together. The model never sees half a turn.
- The system message is never dropped, and neither is the current turn.
- Once a turn has been dropped from a node's prompt, it does not come back on later turns. It is still in the dialog history.

The node does **not** stop as soon as the prompt fits again. It keeps dropping turns until the prompt is down to **75 % of the budget**.

```
flowchart LR
    In["Prompt for this turn"] -->|"fits the budget"| Out["Send to the model"]
    In -->|"over the budget"| Trim["Drop the oldest whole turn"]
    Trim -->|"still above 75 % of the budget"| Trim
    Trim -->|"at or below 75 %"| Out
```

### Why trim to 75 %

Every trim changes the start of the prompt, so the model has to read all the remaining history again. That turn takes noticeably longer than usual.

If the node trimmed only until the prompt fit, it would be just under the budget afterwards. The next line from the player would push it over again, and the node would pay for a full re-read on almost every turn. Trimming to 75 % leaves a quarter of the budget for new conversation, so the node pays for one re-read and then has many cheap turns before the next one. This is **hysteresis**: one level triggers the trim and a lower level ends it, so the prompt doesn't keep crossing the limit back and forth.

With the example above (budget 3,584 tokens), a trim starts when the prompt passes 3,584 tokens and stops at 2,688. That leaves about 900 tokens, several ordinary turns, before the next trim.

The cost of this: after a trim, the node's prompt holds less history than it could. To keep what the dropped turns said, use a rolling summary.

## Rolling summaries keep the thread

Trimming forgets. A [Summarize](https://docs.tryllengine.com/v0.10/reference/nodes/summarize.html) node folds older turns into a short summary **before** they are trimmed:

- When the turns not yet summarized reach **Trigger Tokens**, the node writes a new summary that covers them and the previous summary.
- The latest **Keep Recent Turns** turns are never summarized. They are always replayed word for word.
- On later turns, every Generate, Generate and Speak, and Tool Call node appends the newest summary to its system message, labelled as dialogue data rather than instructions. History then starts after the turns the summary covers.

Because the system message is never trimmed, the summary stays in the prompt while the turns it replaced leave it. Two rules follow:

- **Set Trigger Tokens below the budget of the node that answers**, so the summary is written while the turns are still in the window.
- **Place the Summarize node last**, after the reply the player is waiting on, so the summary's time is not added before that reply.

The full recipe, with sizing numbers, is [Keep long conversations coherent](https://docs.tryllengine.com/v0.10/how-to/keep-long-conversations-coherent.html).

A summary keeps the thread of a scene: where it is, what was agreed, what is still open. Recalling one exact fact much later is a job for [retrieval](https://docs.tryllengine.com/v0.10/concepts/rag.html), for example [a memory the NPC can search](https://docs.tryllengine.com/v0.10/how-to/give-an-npc-searchable-memory.html).

## Pitfalls

- **A long system prompt shrinks the conversation.** The system message is never trimmed, so every token in it comes out of the history budget on every turn. A 3,000-token system prompt in a 4,096-token window leaves room for almost nothing else. If even the system message and the current turn don't fit, the server logs a warning and the model sees no history at all.
- **Max Tokens comes out of the window.** Raising it gives the answer more room and the history less.
- **Each node has its own budget.** In a Tool Call → Generate graph, the two nodes trim independently, so after a long conversation they may not see the same earliest turn.
- **Context Size and the model are fixed for the agent's lifetime.** To change either, create a new agent.

## Seeing it in the editor

- **Turn Inspector → Prompt & output** shows each node's prompt exactly as the model received it: after the summary, history replay, template rendering and trimming. If a turn you expected is missing, it was trimmed or summarized. See [Turn Inspector](https://docs.tryllengine.com/v0.10/reference/turn-inspector.html).
- **Raw JSON → `input.prompt[]`** has the same prompt as data. See [Turn diagnostics JSON](https://docs.tryllengine.com/v0.10/reference/turn-diagnostics-json.html).
- **Export dialog** gives you the full history, including the turns the prompt no longer shows.

## Related

- [How-to: Keep long conversations coherent](https://docs.tryllengine.com/v0.10/how-to/keep-long-conversations-coherent.html)
- [How-to: Run many agents on a small GPU](https://docs.tryllengine.com/v0.10/how-to/run-many-agents-on-a-small-gpu.html)
- [How-to: Estimate the RAM and VRAM your setup needs](https://docs.tryllengine.com/v0.10/how-to/estimate-memory-footprint.html)
- [Summarize node](https://docs.tryllengine.com/v0.10/reference/nodes/summarize.html)
- [Generate node](https://docs.tryllengine.com/v0.10/reference/nodes/generate.html)
- [Reference: Agent Parameters](https://docs.tryllengine.com/v0.10/reference/agent-parameters.html)
- [Concept: Models and Inference Engines](https://docs.tryllengine.com/v0.10/concepts/models-and-inference-engines.html)
- [Concept: Workflows, Graphs, and Nodes](https://docs.tryllengine.com/v0.10/concepts/workflows-graphs-nodes.html)
- [Concept: Tool Calling](https://docs.tryllengine.com/v0.10/concepts/tool-calling.html)
- [Concept: RAG](https://docs.tryllengine.com/v0.10/concepts/rag.html)
