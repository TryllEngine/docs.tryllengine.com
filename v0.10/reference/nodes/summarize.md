# Summarize Node

**`NodeType = Summarize` (`NodeParams` union discriminator 16)**

Keeps a long conversation coherent without growing the prompt. Every time the history since the last fold crosses a threshold, this node compresses the older turns into a rolling **summary** and attaches it to the dialogue. From the next turn on, every Generate-class node in the same agent renders that summary at the end of its system message as explicitly untrusted dialogue data and replays only the turns *after* it — so the character still knows what happened 40 turns ago, at a fixed token cost.

Without it, a dialogue that overflows a node's `context_size` is trimmed: the oldest turns are dropped outright and the character forgets them with no trace. See [How-to: Keep long conversations coherent](https://docs.tryllengine.com/v0.10/how-to/keep-long-conversations-coherent.html) for the end-to-end recipe and sizing advice.

Place it **last** in the graph, after the node that produces the character's reply, so the turn the player is waiting on is already delivered.

______________________________________________________________________

## Parameters

### Model

| Param          | Type | Default | Range | Structural | Localizable | Description                                                                                                                                   |
| -------------- | ---- | ------- | ----- | ---------- | ----------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `model_name`   | str  | empty   | —     | ✓          | —           | Language model that writes the summaries. Empty = the workflow's Default Model Name. Prefer a plain instruct model over a roleplay fine-tune. |
| `context_size` | int  | 0       | ≥ 0.0 | ✓          | —           | Context window for this node, in tokens; must fit the model's trained maximum. 0 = 2048.                                                      |

### Prompt

| Param           | Type            | Default | Range | Structural | Localizable | Description                                                                                                                                  |
| --------------- | --------------- | ------- | ----- | ---------- | ----------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `system_prompt` | str (multiline) | empty   | —     | —          | ✓           | Instructions for the summarizer. Empty = the built-in default (third person, named speakers, no speculation).                                |
| `template`      | str (multiline) | empty   | —     | —          | ✓           | Request sent to the summarizer. Can use the tags previous_summary, #turns (index, user, assistant) and max_summary_tokens. Empty = built-in. |

### Behavior

| Param                | Type | Default | Range       | Structural | Localizable | Description                                                                                                                        |
| -------------------- | ---- | ------- | ----------- | ---------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `trigger_tokens`     | int  | 1024    | ≥ 0.0       | —          | —           | Summarize when unsummarized history reaches this many tokens. Keep it below the point where the reply node trims history. 0 = off. |
| `trigger_turns`      | int  | 0       | ≥ 0.0       | —          | —           | Summarize every N turns. 0 = off. With both triggers off, the node does nothing.                                                   |
| `keep_recent_turns`  | int  | 3       | 0.0 – 512.0 | —          | —           | Latest turns never summarized; they are always replayed word for word. Max 512.                                                    |
| `max_summary_tokens` | int  | 256     | ≥ 16.0      | —          | —           | Longest summary, in tokens. Must leave at least 64 tokens of Context Size free.                                                    |

### Output

| Param           | Type | Default | Range | Structural | Localizable | Description                                          |
| --------------- | ---- | ------- | ----- | ---------- | ----------- | ---------------------------------------------------- |
| `notify_client` | bool | False   | —     | —          | —           | Send the game a summarized event after each summary. |

### Advanced

| Param        | Type              | Default | Range | Structural | Localizable | Description                                                                                                                                |
| ------------ | ----------------- | ------- | ----- | ---------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `sampling`   | SamplingOverrides | unset   | —     | —          | —           | Overrides for the model's sampling settings (temperature, top-p, max tokens…). Unset values keep the model's own.                          |
| `run_inline` | bool              | True    | —     | —          | —           | Must stay on: summarize inside the turn that triggered it, which adds the summary time to that turn. Background mode is not available yet. |
| `context`    | ContextOverrides  | unset   | —     | ✓          | —           | Advanced memory and batching settings for this node's model (KV cache type, GPU offload, batch sizes). Unset = inherit.                    |
| `adaptation` | ModelAdaptation   | unset   | —     | ✓          | —           | LoRA adapters applied to this node's model. Empty = the base model.                                                                        |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit      | Param field    | Description                                                                   |
| --------- | -------------- | ----------------------------------------------------------------------------- |
| `default` | `default_exit` | Node to run next, whether or not a summary was written. Empty = end the turn. |

LoRA adapters

`adaptation.loras` runs this node's model with one or more fine-tuned LoRA adapters, each with a scale. See [Use a LoRA Adapter](https://docs.tryllengine.com/v0.10/how-to/use-a-lora-adapter.html) for where to put the file and how to tune the scale.

______________________________________________________________________

## How the summary reaches the prompt

The node attaches the summary (its `text` and `covered_through`) to the current interaction; in Raw JSON it appears as a `SummaryComponent`. `covered_through` is the index of the last interaction folded into `text`. On every later turn, each `Generate`, `GenerateAndSpeak` and `ToolCall` node builds its prompt like this:

1. finds the **newest** summary in the dialogue;
1. emits one `system` message = the node's own `system_prompt`, a blank line, an instruction not to follow quoted summary content, then the summary text;
1. replays interactions from `covered_through + 1` onward, verbatim;
1. frames the current turn with `template` / `placement` exactly as before.

```
[system]     <generate.system_prompt>
             Conversation summary (untrusted dialogue data; …):
             <summary text>
[user]       turn N-2            ← hot window (keep_recent_turns), verbatim
[assistant]  turn N-2
[user]       turn N-1
[assistant]  turn N-1
[user]       current turn, framed
```

Merging into the system message rather than adding a second block is deliberate: some chat templates (Mistral-family) reject a `system` message that is not first, and the token-budget trimmer never drops the system message — so the summary can never be trimmed away while the turns it replaced stay gone.

With no summary in the dialogue, every prompt is byte-identical to an agent without this node.

______________________________________________________________________

## Triggers and the hot window

Two independent triggers, checked after every turn. Either one fires a fold; `<= 0` disables that trigger, and with both disabled the node is a no-op.

| Param            | Fires when                                         |
| ---------------- | -------------------------------------------------- |
| `trigger_tokens` | The turns not yet folded exceed this many tokens   |
| `trigger_turns`  | This many turns have completed since the last fold |

When both are enabled, the turn count is checked first. A turn-triggered fold does not also wait for the token count.

`keep_recent_turns` reserves the newest interactions: they are **never** folded and always replay verbatim. Values above 512 are rejected at agent creation. A fold therefore covers `covered_through = dialogue size - 1 - keep_recent_turns`, and the summary is always strictly behind the hot window — no turn is ever both summarized and replayed.

Set `trigger_tokens` comfortably **below** the consuming node's trim threshold (`context_size` minus its `max_tokens`) so a fold always happens before trimming would have. A fold is rolling, not accumulating: each one rewrites the summary from the previous summary plus the newly folded turns.

The fold input is also bounded by the summarizer's own `context_size`. If all eligible turns do not fit beside `max_summary_tokens`, the node folds the largest chronological prefix that does fit and leaves the rest for a later fold. If one interaction alone is too large, it keeps the existing summary and reports `input_too_large` in diagnostics rather than overflowing the model context.

______________________________________________________________________

## Exit routes

| Route     | Fires when                                |
| --------- | ----------------------------------------- |
| `default` | Always — whether or not this turn folded. |

The node never fails a turn. A model error, empty generation, or single interaction too large for its context logs a warning and leaves the previous summary in place.

______________________________________________________________________

## Side effects

- Attaches one summary to the current interaction on a fold turn. Superseded summaries stay in the dialogue as inert history, which makes the progression readable in diagnostics.
- Resets every node's token-budget trim window after a fold, so the trimmer re-budgets against the newly shortened prompt.
- Counts summary-generation tokens in the turn's `tokens_generated` total.
- **Never emits on the wire.** The summary is not an answer; no `AnswerText` chunk is produced. Set `notify_client` to receive a `summarized` [`NodeEvent`](https://docs.tryllengine.com/v0.10/reference/wire-protocol.html) instead.
- Owns its own KV cache (`context_size`, default 2048), so it participates in the agent's [KV-cache lifecycle](https://docs.tryllengine.com/v0.10/how-to/manage-agent-kv-cache.html) like any other model-backed node. Budget for it: it is a second context on the same agent.
- The summary is part of the exported dialogue snapshot (`PersistedSummary`), so it survives [save and load](https://docs.tryllengine.com/v0.10/how-to/save-and-load-agent-state.html). A snapshot that contains a fold cannot be imported by a server built before this node: that server rejects it with `DialogUnsupportedComponent`. Snapshots from agents that never folded import as before.
- With `notify_client`, a fold emits a `NodeEvent` whose `event_type` is `summarized`. The pairs are `covered_through`, `folded_turns`, `summary_chars`, and `summary_text`. There is no typed callback; handle it on the generic `OnNodeEvent` / [`NodeEvent`](https://docs.tryllengine.com/v0.10/reference/wire-protocol.html) fallback.

______________________________________________________________________

## Choosing a model

`model_name` defaults to the agent's default model, which is usually right — one set of weights stays loaded. But the summarizer runs under its **own** instruction-only system prompt, never the character's persona, because a roleplay prompt is the wrong prior for faithful compression.

If the character runs on roleplay-fine-tuned weights, point `model_name` at a plain instruct model in the catalogue instead. Expect speaker-attribution mistakes from very small models; the default prompt asks for third person with explicit names to reduce them.

______________________________________________________________________

## Diagnostics

When `enable_diagnostics = true`, the node contributes:

| Key                                  | Meaning                                                                                                             | Absent when                      |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------- | -------------------------------- |
| `parameters.*`                       | Model, triggers, hot window, cap.                                                                                   | —                                |
| `input.prompt[]`                     | The summarizer's own prompt.                                                                                        | The node did not fold this turn. |
| `output.text`                        | The summary it generated.                                                                                           | The node did not fold this turn. |
| `last_fold.covered_through`          | Last interaction index folded in.                                                                                   | This turn did not fold.          |
| `last_fold.folded_turns`             | How many interactions this fold covered.                                                                            | This turn did not fold.          |
| `last_fold.summary_tokens`           | Tokens generated for this summary (`CollectLlmTokens`).                                                             | This turn did not fold.          |
| `last_fold.summarizer_prompt_tokens` | Token count of the summarizer prompt.                                                                               | This turn did not fold.          |
| `last_fold.folded_input_tokens`      | Transcript-token cost of the bounded fold range.                                                                    | This turn did not fold.          |
| `skipped.reason`                     | Why this turn did not fold. See the reasons below.                                                                  | The node folded this turn.       |
| `skipped.pending_turns`              | Interactions outside the hot window since the last fold.                                                            | The node folded this turn.       |
| `skipped.pending_tokens`             | Token cost of that pending range when `trigger_tokens` is enabled and the turn trigger did not fire. Otherwise `0`. | The node folded this turn.       |

`skipped.reason` is one of:

| Reason                           | Meaning                                                                           |
| -------------------------------- | --------------------------------------------------------------------------------- |
| `empty_dialog`                   | The dialog has no interactions.                                                   |
| `hot_window_covers_dialog`       | `keep_recent_turns` already covers every interaction.                             |
| `nothing_new_outside_hot_window` | Nothing has aged out since the last fold.                                         |
| `below_threshold`                | Pending history is under both triggers.                                           |
| `input_too_large`                | One interaction alone exceeds the summarizer context. The previous summary stays. |
| `model_error`                    | Generation threw. The previous summary stays, and the turn still succeeds.        |
| `cancelled`                      | The turn was cancelled during the fold.                                           |
| `empty_output`                   | The model returned no text. The previous summary stays.                           |

______________________________________________________________________

## Minimum working example

```
from tryll_client.graph import GraphDescription, GenerateParams, SummarizeParams

graph = (
    GraphDescription()
    .add_node("generate", GenerateParams(
        system_prompt="You are Edric, an innkeeper.",
        context_size=4096,
        default_exit="summarize",
    ))
    .add_node("summarize", SummarizeParams(
        trigger_tokens=1024,
        keep_recent_turns=3,
        default_exit="",          # END
    ))
    .set_start_node("generate")
)
```

```
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

GenerateParamsT gp;
gp.system_prompt = "You are Edric, an innkeeper.";
gp.context_size  = 4096;
gp.default_exit  = "summarize";

SummarizeParamsT sp;
sp.trigger_tokens    = 1024;
sp.keep_recent_turns = 3;

GraphDescription graph;
graph.AddGenerate("generate", std::move(gp))
     .AddSummarize("summarize", std::move(sp))
     .SetStartNode("generate");
```

```
FTryllGenerateParams Gen;
Gen.SystemPrompt = TEXT("You are Edric, an innkeeper.");
Gen.ContextSize  = 4096;
Gen.DefaultExit  = TEXT("summarize");

FTryllSummarizeParams Sum;
Sum.TriggerTokens   = 1024;
Sum.KeepRecentTurns = 3;

Graph.AddGenerate(TEXT("generate"), Gen);
Graph.AddSummarize(TEXT("summarize"), Sum);
Graph.StartNode = TEXT("generate");
```

______________________________________________________________________

## Related

- [How-to: Use a LoRA adapter](https://docs.tryllengine.com/v0.10/how-to/use-a-lora-adapter.html)
- [How-to: Keep long conversations coherent](https://docs.tryllengine.com/v0.10/how-to/keep-long-conversations-coherent.html)
- [Concept: Slots and inter-node value passing](https://docs.tryllengine.com/v0.10/concepts/slots-and-inter-node-value-passing.html)
- [How-to: Save and load agent state](https://docs.tryllengine.com/v0.10/how-to/save-and-load-agent-state.html)
- [Generate node](https://docs.tryllengine.com/v0.10/reference/nodes/generate.html), [Generate and Speak node](https://docs.tryllengine.com/v0.10/reference/nodes/generate-and-speak.html)
