# Design an Intent-Classifier Prompt

How to write the `system_prompt` and `intents` array of a [`ClassifyIntentLLM`](https://docs.tryllengine.com/v0.9/reference/nodes/classify-intent-llm.html) node so a small on-device model routes the player's line to the right branch — and defers when it genuinely cannot tell.

This node's prompt is unlike every other prompt in Tryll. It is a **Mustache template** that must reference `{{intents_block}}`. Empty or unparsable templates fail `CreateAgent` and `ChangeAgentParam`. `{{letters}}` is optional and usually not worth including. That is the first thing to get right, and it is covered in [The template contract](#the-template-contract).

How claims are marked

- **Tested** — measured, on either the [immersion-guard corpus](https://docs.tryllengine.com/v0.9/how-to/build-an-immersion-guard.html) or the published literature cited inline.
- **Hint** — a real effect, but narrow: one model, one label set, or a design rule rather than a score delta.
- **No effect** — we tried it; do not spend time here.

______________________________________________________________________

## What the prompt is actually for

`ClassifyIntentLLM` runs the model **once**, reads the logits at the final position of the prefilled prompt, and softmaxes them over the token ids for `A`, `B`, `C`, … No text is generated. No sampler runs. The decision is made before the model would have written a single word.

So the prompt has exactly three jobs:

1. **Define the label space** in plain language.
1. **Anchor the model to the latest user turn**, so multi-turn history does not become a re-classification target.
1. **Position the tokenizer** so the next token would be a bare letter — no preamble, no punctuation, no markdown.

Anything that hopes to shape behaviour *after* the first token has no effect at all. "Think step by step", "explain your reasoning", "answer in JSON" — those tokens are paid for in prefill and then discarded unread. **Tested**: this follows directly from the mechanism, not from taste.

If you need the model to reason before committing, this is the wrong node. Route `not_found` to a heavier downstream step instead — see [Two ways to say "I don't know"](#two-ways-to-say-i-dont-know).

______________________________________________________________________

## The template contract

`system_prompt` is rendered as a [Mustache](https://mustache.github.io/) template against a context with two variables:

| Variable            | Required | Renders to                                                                                                        |
| ------------------- | -------- | ----------------------------------------------------------------------------------------------------------------- |
| `{{intents_block}}` | yes      | The letter-prefixed label list, one line per `intents` prompt: `A. <first description>\nB. <second description>…` |
| `{{letters}}`       | no       | The candidate letters, comma-joined: `A, B, C`                                                                    |

Start from this template:

```
You are an intent classifier. Pick the best label for the user's LATEST message,
using the conversation as context for pronouns and references.

Labels:
{{intents_block}}

Respond with a single letter. No other text.
```

Four pieces in there are load-bearing:

- **`Labels:`** — the caption word is a cue, not decoration. Changing it to `Options:` cost 3.4 pp; deleting it cost 2.3 pp. **Tested** (label-rendering bake-off, 5 models × 3 tasks).
- **`{{intents_block}}`** — this is what keeps letter `A` in the prompt aligned with `intents[0]`. The node scores `A`/`B`/`C` from the array; if you write the list by hand they can drift. Empty, unparsable, or missing `{{intents_block}}` (padded and triple-brace forms count) fails `CreateAgent` and `ChangeAgentParam`.
- **"LATEST message"** — without it, the model sometimes picks a label that fits an earlier turn. Short, cheap, consistently helpful. **Hint.**
- **"Respond with a single letter. No other text."** — the last thing before the chat template's assistant opener, so it sets what the model expects to emit next. Deleting the sentence cost 2.5–2.9 pp, and up to 12 pp on one model; a prompt that ends on the label list nearly doubles letter-position skew. **Tested** (same bake-off).

`{{letters}}` is still substituted if you write it. Restating `"(one of A, B, C)"` is redundant with the block and slightly worse-calibrated; leave it out. **Tested.**

Required Mustache

Empty or unparsable `system_prompt`, or a template that omits `{{intents_block}}`, fails `CreateAgent` and `ChangeAgentParam`. There is no built-in default template. Check the rendered prompt in `input.prompt[]` under [turn diagnostics](https://docs.tryllengine.com/v0.9/reference/turn-diagnostics-json.html) after any edit — that is the only place the substitution is visible.

This is not the `Generate` Mustache context

[`Generate`](https://docs.tryllengine.com/v0.9/reference/nodes/generate.html) and [`Transform`](https://docs.tryllengine.com/v0.9/reference/nodes/transform.html) templates get `user_message`, `slot.<name>`, `var.<name>`, `instructions`, and `knowledge` — see [Use Mustache Templates](https://docs.tryllengine.com/v0.9/how-to/use-mustache-templates.html). **None of those resolve here.** A classifier system prompt has `intents_block` and optional `letters` and nothing else; the user's message arrives as its own user turn, added by the projection.

______________________________________________________________________

## How a letter becomes an intent id

You author an `intents` array of id/prompt pairs. **Their order is the contract.**

```
from tryll_client.graph import ClassifyIntentEntry

intents = [
    ClassifyIntentEntry(id="greeting",
        prompt="Greeting, introduction, or pleasantries only — not case questions"),
    ClassifyIntentEntry(id="confess",
        prompt="Mrs Hollis saw you leaving the study with blood on your hands"),
    ClassifyIntentEntry(id="when_saw_body",
        prompt="Inspector asks when or at what time the butler first saw the body"),
    ClassifyIntentEntry(id="other",
        prompt="Anything that does not match the labels above"),
]
```

| Index | Letter shown to the model | Text shown to the model (`prompt`)            | Attached on a win (`id`) |
| ----- | ------------------------- | --------------------------------------------- | ------------------------ |
| 0     | `A`                       | Greeting, introduction, or pleasantries only… | `greeting`               |
| 1     | `B`                       | Mrs Hollis saw you leaving the study…         | `confess`                |
| 2     | `C`                       | Inspector asks when or at what time…          | `when_saw_body`          |
| 3     | `D`                       | Anything that does not match…                 | `other`                  |

The key consequence: **the id is never shown to the model.** It is downstream wiring only — the string that lands in the `IntentionComponent` for [`IntentToInstruction`](https://docs.tryllengine.com/v0.9/reference/nodes/intent-to-instruction.html) or a [`Branch`](https://docs.tryllengine.com/v0.9/reference/nodes/branch.html) to route on. So ids can stay machine-shaped and stable, while *all* of the semantic work happens in `prompt`. Renaming an id breaks your graph; rewriting a description breaks nothing.

Why letters rather than the ids themselves? Because `when_saw_body` tokenizes into three to five pieces, and you cannot read a multi-token label off a single logit. Letters are single tokens in essentially every modern tokenizer, and instruction-tuned models have seen enormous amounts of `A`/`B`/`C`/`D` multiple-choice data.

### Constraints the server enforces at agent creation

| Rule                                                          | Failure if violated                                                   |
| ------------------------------------------------------------- | --------------------------------------------------------------------- |
| 2–26 labels                                                   | `label count must be between 2 and 26`                                |
| Each `id` and `prompt` is non-empty after trim                | empty id / empty prompt                                               |
| Ids are unique                                                | `intents contains duplicate ids`                                      |
| Each letter `A`…`<last>` is a single token for *this* model   | `label letter 'C' does not tokenize to a single token for this model` |
| `not_found_intent`, if set, is one of the ids or `#uncertain` | `not_found_intent '…' must be one of the intents ids or '#uncertain'` |

`CreateAgent` fails — these are not runtime surprises.

______________________________________________________________________

## Writing the descriptions

Each entry's `prompt` is where you spend your tuning budget. Everything else is scaffolding.

**Write what the user says, not what the system does.** The model routes on the text after the `A.` marker. `Inspector asks when or at what time the butler first saw the body` is doing real work; `when_saw_body` would be doing none.

**Make them mutually exclusive.** Overlap between two descriptions is the single most common cause of low `margin` in production: the model splits probability mass across both and your margin gate rejects everything. When you find a confusion pair — and you will — the fix is almost always a negation clause on one of them: `Greeting or introduction only — not case questions`. **Tested** (Arora et al., [arXiv:2410.01627](https://arxiv.org/abs/2410.01627), on intent scope and out-of-scope behaviour).

**Make them collectively exhaustive, with an explicit catch-all.** Without an `other` / `anything else` entry, the model forces every out-of-scope utterance onto an in-scope label.

**Fold per-label rules into the description.** A negation clause, a fact list, or a short "not a when-question" rule belongs on that label's `prompt`, not in pseudo-HTML tags above the block. Placement of the rules is free; *deleting* them cost 4.8 pp. **Tested** (label-rendering bake-off). Keep each entry readable — long multi-clause text adds prefill without a free accuracy win.

**Match scope to label count.** For dialogue gating (2–8 labels), prefer specific scopes plus one broad catch-all. Past roughly 10–15 labels, single-shot LLM classification degrades and you want an embedding pre-filter in front of it — or [`ClassifyIntent`](https://docs.tryllengine.com/v0.9/reference/nodes/classify-intent.html) instead. **Tested** (Arora et al.).

**Use your own failures as examples.** `ClassifyIntentLLM` has no few-shot parameter, so worked examples go directly into the `system_prompt` above the `Labels:` block. On the immersion-guard corpus, adding five examples — one per category the model was actually getting wrong — bought about 5 points of recall at the same false-positive rate. Generic examples did not help. **Tested**; see [Build an immersion guard](https://docs.tryllengine.com/v0.9/how-to/build-an-immersion-guard.html).

Never seed few-shot examples from misclassified samples: you would be teaching the wrong boundary.

______________________________________________________________________

## Order the labels deliberately

Models carry strong, **model-specific** positional bias over the answer letter. Zheng et al. ([arXiv:2309.03882](https://arxiv.org/abs/2309.03882)) measured this across model families; the immersion-guard work saw a case where adding irrelevant context made a classifier answer with the *first* label on 29 of 38 good lines. **Tested.**

What follows from that:

- **Put the catch-all last.** Its probability is the one you most want to read honestly, and it should win on content rather than position.
- **Try both orders on a dev set before you ship.** Order alone moved pooled accuracy by 2.7 pp and one model/task cell by 65 pp — larger than every formatting choice measured. **Tested** (label-rendering bake-off).
- **Two orderings of the same label set are not equivalent.** If shuffling the order moves your accuracy much, your descriptions are not pulling their weight.
- **Re-tune when you change classifier models.** Bias direction differs between Llama, Qwen, Mistral and Phi. A byte-identical prompt will score differently.

______________________________________________________________________

## Conversation history

`history_turns` (default `2`, range `0`–`64`) controls how many past user/assistant pairs are projected before the current message. The in-progress turn's own assistant reply is deliberately excluded — that is the position whose logits we score.

| Situation                                                      | Setting                                       |
| -------------------------------------------------------------- | --------------------------------------------- |
| Pronouns and back-references ("and when did *you* see *him*?") | `2` is usually enough                         |
| Single-turn commands, or screening each line independently     | `0` — saves prefill and removes a bias source |

The non-obvious failure mode: **past assistant replies bias classification.** If the NPC just mentioned a time, the next player line is more likely to be classified as the timing intent, despite your "LATEST message" instruction. If history is hurting, fix descriptions first, reduce `history_turns` second.

History always replays `user_message`

If you point `input` at a different slot, that re-selects the text for the **current** turn only. History turns always replay the plain `user_message` slot.

______________________________________________________________________

## Threshold, margin, and the confidence you actually get

`top_prob` is a softmax over **candidate letters only**:

```
P(label_i) = exp(logit_i) / Σ_j exp(logit_j)    for j ∈ {A, B, C, …}
```

That answers *"among the labels I may pick, how much mass does each get?"* It does **not** answer *"how likely is this to be correct?"* A model can put 0.95 on the wrong label, especially under position bias or description overlap. Use `top_prob` comparatively, and tune on a held-out set rather than on intuition.

| Param       | Gate                                                           | Default |
| ----------- | -------------------------------------------------------------- | ------- |
| `threshold` | `top_prob` must reach it                                       | `0.5`   |
| `margin`    | `top_prob − second_prob` must reach it; `0` disables the check | `0.15`  |

`margin` is the one that catches ambiguity: `0.55` against a runner-up of `0.40` is much weaker than `0.55` against `0.10`, and only the margin gate can tell them apart.

Do not leave `threshold` at 0.5

On the immersion-guard corpus, setting `threshold` from the model's measured confidence distribution instead of the default took wrongly-blocked player lines from **16.7% to 2.6%** — more than every prompt change tried, combined. **Tested**; the method is in [Tuning the threshold](https://docs.tryllengine.com/v0.9/how-to/build-an-immersion-guard.html#tuning-the-threshold).

Tune in this order: descriptions → `history_turns` → `threshold` and `margin` together. Threshold tuning cannot fix label-scope overlap; if you are pushing `threshold` to 0.8 to suppress mis-routing, the descriptions are the problem.

### Two ways to say "I don't know"

1. **An explicit catch-all label.** The model picks it, the node exits `found`, and your catch-all handler runs. Use it when you always want *some* intent tag attached.
1. **The `not_found` route.** Low confidence rejects classification entirely. Use it when mis-routing is worse than deferring — e.g. classifying an unrelated question as a scripted beat would corrupt the scene, while falling through to retrieval produces a real answer.

`not_found_intent` bridges the two: name one of your intent ids, and when that label wins, the node exits `not_found` instead of `found`. That is how the immersion guard makes its *allow* label the fall-through path while every other label routes to deflection.

______________________________________________________________________

## Read the decision in diagnostics

With `enable_diagnostics` on, the node reports the full picture — and this is the **only** place it is visible, since the [Turn Inspector](https://docs.tryllengine.com/v0.9/reference/turn-inspector.html) has no classification block yet. Open Raw JSON:

| Key                                                                | Use it to                                                                                                |
| ------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------- |
| `input.prompt[]`                                                   | Confirm `{{intents_block}}` substituted (and `{{letters}}`, if you used it)                              |
| `output.classification.probs[]`                                    | Read the whole ranking — positionally aligned with `parameters.intents_ids[]`                            |
| `output.classification.top_prob` / `second_prob` / `margin_result` | Compare against `parameters.threshold` / `parameters.margin`                                             |
| `output.classification.predicted_letter`                           | The winning letter                                                                                       |
| `output.classification.not_found_reason`                           | `empty_message`, `below_threshold`, `below_margin`, `sink_intent`, `score_failed`, `score_size_mismatch` |

Setting `notify_client` to `OnFound` or `Always` pushes the same numbers to the client as an `intent_llm_classified` event, including one `<label>.prob` per entry — which is how you collect a confidence distribution to tune `threshold` against. See [Use the Agent Log](https://docs.tryllengine.com/v0.9/how-to/use-agent-log.html).

______________________________________________________________________

## An iteration loop that works

1. **Baseline.** Default template, one-line description per intent, `history_turns = 0`, `threshold = 0.5`, `margin = 0`.
1. **Build a dev set.** 20–50 in-scope lines per intent, plus 20–50 deliberate boundary and out-of-scope lines. The boundary cases matter most.
1. **Run with `notify_client = Always`** and capture `intent`, `top_prob`, `second_prob`, and the rendered prompt for every line.
1. **Group failures by (expected, predicted) pair** and sort by frequency. The top two or three pairs hide most of the remaining gains.
1. **Write adversarial lines** for each top confusion pair and confirm they reproduce the failure.
1. **Revise descriptions** — negation clauses, tighter scope. Do *not* rename intent ids; downstream wiring depends on them.
1. **Tune `history_turns`**, then `threshold` and `margin`, against your routing policy.
1. **Re-run the whole dev set after any template edit.** Small wording and formatting changes swing classification accuracy by a few points (Sclar et al., [arXiv:2310.11324](https://arxiv.org/abs/2310.11324)). Change one thing at a time. **Tested.**

______________________________________________________________________

## Anti-patterns

- [ ] **Empty or incomplete `system_prompt`.** Must parse and reference `{{intents_block}}`, or `CreateAgent` fails.
- [ ] **Prompt ends on the label list.** Keep *"Respond with a single letter. No other text."* — deleting that sentence cost 2.5–2.9 pp.
- [ ] **`Options:` (or no caption) instead of `Labels:`.** Caption word is load-bearing.
- [ ] **XML or JSON for the label block.** Costs calibration and 12–26 prefill tokens per turn; `A. <description>` is already on the plateau.
- [ ] **Chain-of-thought or "explain your answer" instructions.** Never executed.
- [ ] **Asking for JSON.** Shifts first-token mass toward `{` and defeats the mechanism.
- [ ] **`Answer:` or `The intent is:` as the closing line.** You may be reading the logit for that word instead of a letter.
- [ ] **`{{user_message}}`, `{{slot.x}}` or `{{var.x}}` in the template.** Not in this context; they render empty. The user's message is added as its own turn.
- [ ] **Embedding the user's message in the system prompt.** The projection already adds it.
- [ ] **Commas inside a legacy CSV description.** Silently becomes an extra label. Prefer the `intents` array.
- [ ] **Overlapping or copy-pasted descriptions.** The #1 cause of low margin.
- [ ] **The most common intent at `A`** without checking position bias.
- [ ] **Tuning `threshold` before fixing description overlap.** Treats the symptom.
- [ ] **Treating `top_prob` as absolute confidence.** It is a comparative softmax over letters.
- [ ] **More than ~10–15 labels on a 1B-class model** without a pre-filter.

______________________________________________________________________

## Related

- [Reference: Classify Intent (LLM) node](https://docs.tryllengine.com/v0.9/reference/nodes/classify-intent-llm.html) — every parameter, exit, and diagnostic key
- [How-to: Classify intent without a knowledge base](https://docs.tryllengine.com/v0.9/how-to/classify-intent-without-a-knowledge-base.html) — the end-to-end graph, in all four clients
- [How-to: Build an immersion guard](https://docs.tryllengine.com/v0.9/how-to/build-an-immersion-guard.html) — two measured classifiers, and the threshold-tuning method
- [How-to: Build an intent-driven NPC](https://docs.tryllengine.com/v0.9/how-to/build-intent-driven-npc.html) — the embedding-based alternative, when you have labelled examples
- [How-to: Design an NPC prompt](https://docs.tryllengine.com/v0.9/how-to/design-an-npc-prompt.html) — the prompt on the other side of the graph
- [How-to: Change agent parameters at runtime](https://docs.tryllengine.com/v0.9/how-to/change-agent-parameters.html) — `system_prompt`, `threshold` and `margin` are mutable; the label lists are not
- [Concept: Constrained output](https://docs.tryllengine.com/v0.9/concepts/constrained-output.html) — when to reach for a grammar instead
