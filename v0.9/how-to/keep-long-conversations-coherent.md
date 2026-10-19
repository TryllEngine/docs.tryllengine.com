# Keep Long Conversations Coherent

Stop a character forgetting the start of a long conversation, without paying for an ever-growing prompt.

Prerequisites

- A working chat agent — see [Build a Chat Agent with a Graph](https://docs.tryllengine.com/v0.9/how-to/build-chat-agent-with-graph.html).
- A conversation long enough to matter: roughly, more turns than `context_size` divided by the size of one turn-pair.

## The problem

Every model-backed node owns a fixed context window. When the projected prompt outgrows it, Tryll **trims**: the oldest turns are dropped from the prompt and never come back. Nothing fails, nothing is logged to the player, and the character simply stops knowing what it was told.

You can see it happen in the server log:

```
[TokenBudgetProjectionStrategy] Windowed prompt: 3891 tokens, 14 messages (_windowStart=6)
```

`_windowStart` only ever advances. Once it passes a turn, that turn is gone.

## The fix

Add a [`Summarize`](https://docs.tryllengine.com/v0.9/reference/nodes/summarize.html) node at the end of the graph. It folds the turns that are about to age out into a rolling summary, which every Generate-class node in the same agent then renders at the end of its system message.

```
flowchart LR
    generate["GenerateAndSpeak<br>generate"]
    summarize["Summarize<br>summarize"]
    generate -- "default" --> summarize
    summarize -- "default" --> END
```

## Steps

### 1. Put the node last

Place it after the node that produces the player-facing reply. The reply is already delivered by the time the summarizer runs, so the player sees the answer at normal speed; only the turn that triggers a fold costs extra.

```
from tryll_client.graph import GraphDescription, GenerateParams, SummarizeParams

graph = (
    GraphDescription()
    .add_node("generate", GenerateParams(
        system_prompt=persona,
        context_size=4096,
        default_exit="summarize",
    ))
    .add_node("summarize", SummarizeParams(
        trigger_tokens=1024,
        keep_recent_turns=3,
        default_exit="",
    ))
    .set_start_node("generate")
)
```

In the workflow asset, add a node of type **Summarize**, then set the Generate node's **Default Exit** to it and leave the Summarize node's own **Default Exit** empty (END).

### 2. Size the trigger against the consuming node

The summarizer must fire **before** the trimmer would. Work it out from the node that consumes the history:

| Quantity                  | Where it comes from                                        |
| ------------------------- | ---------------------------------------------------------- |
| Context window            | The consuming node's `context_size`                        |
| Reserved for generation   | Its `max_tokens`                                           |
| Fixed cost                | System prompt + any retrieved knowledge + template framing |
| **Head-room for history** | Context − generation − fixed cost                          |

Set `trigger_tokens` to roughly **60–70%** of that head-room. Example, for a 4096-token node with a 1024-token system prompt, ~1000 tokens of retrieval and 512 reserved for generation: head-room is about 1500 tokens, so `trigger_tokens = 1024` is a good starting point.

Prefer `trigger_tokens` for production. `trigger_turns` exists for tests and for authored pacing where "every N exchanges" is easier to reason about than tokens.

### 3. Keep a verbatim hot window

`keep_recent_turns` (default 3) is the number of most recent exchanges that are never folded. They always replay word for word.

Raise it when the character needs exact recall of the immediate scene — a negotiation where the last offer matters, a puzzle where wording matters. Lower it to squeeze more head-room out of a small window. Below 2 the character starts paraphrasing the exchange that just happened, which reads as inattentive.

### 4. Write the prompt for compression, not for prose

The node ships a default `system_prompt` and `template` that ask for third person, named speakers, short verbatim quotes of what the player stated or promised, and a hard length cap. Override them when the genre needs it, and keep these rules:

- **Ask it to merge and shorten older points**, not just append new ones. Without this, the summary grows until `max_summary_tokens` truncates it mid-sentence.
- **Name the speakers.** Small models confuse who said what more than they invent facts.
- **Cap the length in the prompt as well as in `max_summary_tokens`.** The parameter is a hard stop that cuts mid-word; the prompt is what makes the model land inside it.
- **Forbid speculation.** "Record only what was said" measurably reduces invented detail on 3–4B models.
- **Treat the transcript as data, not instructions.** The built-in prompt and downstream projection both say this explicitly so quoted player text cannot silently become higher-priority instructions after a fold.

### 5. Watch it work

Set `notify_client = true` and handle the `summarized` [`NodeEvent`](https://docs.tryllengine.com/v0.9/reference/nodes/summarize.html#side-effects) on the generic node-event callback: it carries `covered_through`, `folded_turns`, `summary_chars` and the summary text. During development, printing that text beside the transcript is the fastest way to see whether your prompt is producing something the character can actually use.

With `enable_diagnostics = true`, the turn's `debug_info` also carries the summarizer's own prompt and output under its node entry, and the `SummaryComponent` appears in `interaction.components[]`.

## Budget the second context

The node owns its own KV cache. At the default `context_size = 2048` that is a real, fixed VRAM cost on top of the character's own context — on an 8B model, roughly 135 MB. On a tight GPU:

- Share the weights: leave `model_name` empty so the summarizer uses the agent's default model rather than loading a second one.
- Keep `context_size` at 2048 unless a single fold's backlog needs more room. Each fold sees the previous summary plus the largest chronological prefix of pending turns that fits beside `max_summary_tokens`.
- Keep `max_summary_tokens` at least 64 tokens below `context_size`. Tryll rejects configurations that leave no prompt room and, for large backlogs, folds the largest chronological prefix that fits.
- Let the [KV-cache lifecycle](https://docs.tryllengine.com/v0.9/how-to/manage-agent-kv-cache.html) evict it between folds if the agent is idle for long stretches.

## What this does not do

A summary preserves the **thread** of a conversation — where the scene is, what was agreed, what is unresolved. It is lossy about detail by design, and every fold compounds that loss slightly.

If the character must recall an exact fact many hours later ("what was the name of the merchant in the first village?"), that is a job for retrieval over stored facts, not for a summary. Use [`Retrieve`](https://docs.tryllengine.com/v0.9/reference/nodes/retrieve.html) against a knowledge base for those, and let the summary carry the narrative.

## Related

- [Summarize node reference](https://docs.tryllengine.com/v0.9/reference/nodes/summarize.html)
- [How-to: Save and load agent state](https://docs.tryllengine.com/v0.9/how-to/save-and-load-agent-state.html)
- [How-to: Run many agents on a small GPU](https://docs.tryllengine.com/v0.9/how-to/run-many-agents-on-a-small-gpu.html)
- [How-to: Manage an agent's KV cache](https://docs.tryllengine.com/v0.9/how-to/manage-agent-kv-cache.html)
