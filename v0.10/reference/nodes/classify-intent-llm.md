# Classify Intent (LLM)

The Classify Intent (LLM) node maps a user message to a discrete **intent label** using a small language model's **first-token logprob** over a closed set of letter candidates (`A`, `B`, `C`, …). Each letter maps positionally to an entry in `intents`. It writes the decision to the slot `output_name` (empty = the node's name), so a [Branch](https://docs.tryllengine.com/v0.10/reference/nodes/branch.html) can test it. On success it also attaches an `IntentionComponent` — the same routing-only component produced by [Classify Intent](https://docs.tryllengine.com/v0.10/reference/nodes/classify-intent.html) — so downstream [IntentToInstruction](https://docs.tryllengine.com/v0.10/reference/nodes/intent-to-instruction.html) wiring is unchanged.

`NodeType`: `ClassifyIntentLLM`.

## How it works

1. Projects the dialog via `ClassifyIntentLLMProjectionStrategy`:
1. Stable **system** message — `system_prompt` rendered as a Mustache template against the pre-built label list; see [The system prompt](#the-system-prompt).
1. Up to `history_turns` past **user** / **assistant** pairs for coreference (history turns always replay the `user_message` slot).
1. Current turn's `input`-resolved slot as the **user** message (empty = `user_message`).
1. Prefills the classifier model (`Sync()`); reuses KV-cache prefix across turns when the prompt tail is append-only.
1. Reads logits at the final position and softmaxes over the single-token ids for `A`, `B`, `C`, …
1. If `top_prob ≥ threshold` and `(top − second) ≥ margin`, attaches `IntentionComponent` for the winning id and returns `found`.
1. Writes the output slot on every path — see [Output slot](#output-slot).

No text is generated; no sampling.

## Output slot

The node writes one turn-local slot, `slot.<output_name>`, whichever exit it takes. Empty `output_name` uses the node's own name.

| Exit        | Slot value                                                                                   |
| ----------- | -------------------------------------------------------------------------------------------- |
| `found`     | The winning intent id.                                                                       |
| `not_found` | `not_found_intent` — an intent id, the reserved `#uncertain`, or an empty string when unset. |

`not_found_intent` is written on every `not_found` reason: `below_threshold`, `below_margin`, `sink_intent`, `empty_message` and scoring failures. This is the same rule as the `uncertain_choice` of a `Choice` attribute on [Classify Attributes (LLM)](https://docs.tryllengine.com/v0.10/reference/nodes/classify-attributes-llm.html). Because the slot is always written, a `Branch` placed after either exit can read it.

## The system prompt

`system_prompt` is a [Mustache](https://mustache.github.io/) template rendered against a context with two variables:

| Variable            | Required | Renders to                                                             |
| ------------------- | -------- | ---------------------------------------------------------------------- |
| `{{intents_block}}` | yes      | The label list, one line per intent prompt: `A. <first>\nB. <second>…` |
| `{{letters}}`       | no       | The candidate letters, comma-joined: `A, B, C`                         |

`system_prompt` is required. Empty, unparsable, or missing `{{intents_block}}` (padded and triple-brace forms count) fails `CreateAgent` and `ChangeAgentParam`. `{{letters}}` is still substituted if you write it; leave it out unless you have a reason. There is no built-in default template.

This is **not** the [`Generate`](https://docs.tryllengine.com/v0.10/reference/nodes/generate.html) / [`Transform`](https://docs.tryllengine.com/v0.10/reference/nodes/transform.html) Mustache context: `user_message`, `slot.<name>`, `var.<name>`, `instructions` and `knowledge` do not resolve here. The classified text arrives as its own user turn, added by the projection.

A minimal working template:

```
You are an intent classifier. Pick the best label for the user's LATEST message,
using the conversation as context for pronouns and references.

Labels:
{{intents_block}}

Respond with a single letter. No other text.
```

With three `intents` entries whose prompts are "asks the price", "wants to haggle", and "anything else", the model receives:

```
You are an intent classifier. Pick the best label for the user's LATEST message,
using the conversation as context for pronouns and references.

Labels:
A. asks the price
B. wants to haggle
C. anything else

Respond with a single letter. No other text.
```

Intent ids are never rendered — they supply the label attached on a win, for downstream routing only. For what to write in the template and how to order the labels, see [Design an intent-classifier prompt](https://docs.tryllengine.com/v0.10/how-to/design-an-intent-classifier-prompt.html).

## Context window

Leave `context_size` at `0` to use this node's **2048-token** default. `0` does not fall through to the model variant or the server `default_n_ctx` (only Generate and Generate and Speak inherit those). That is enough for typical classifier prompts with a few history turns and is far cheaper than the 8192 Generate default. The `context` table is an advanced structural override (`kv_cache_type`, veto-only `offload_kqv`, escape-hatch batch sizes). Do not set `q4_0` KV expecting a VRAM win at a right-sized window — measured savings show up only at large `n_ctx` (around 8192).

## Parameters

### Input

| Param           | Type | Default | Range      | Structural | Localizable | Description                                                                                       |
| --------------- | ---- | ------- | ---------- | ---------- | ----------- | ------------------------------------------------------------------------------------------------- |
| `input`         | str  | empty   | —          | ✓          | —           | Text to classify: the player's message or an earlier node's output. Empty = the player's message. |
| `history_turns` | int  | 2       | 0.0 – 64.0 | —          | —           | How many earlier dialog turns the classifier sees. 0 = the current message only.                  |

### Model

| Param          | Type | Default   | Range | Structural | Localizable | Description                                                                              |
| -------------- | ---- | --------- | ----- | ---------- | ----------- | ---------------------------------------------------------------------------------------- |
| `model_name`   | str  | empty     | —     | ✓          | —           | Language model that scores the intents. Empty = the workflow's Default Model Name.       |
| `context_size` | int  | 0 -> 2048 | ≥ 0.0 | ✓          | —           | Context window for this node, in tokens; must fit the model's trained maximum. 0 = 2048. |

### Prompt

| Param           | Type                      | Default    | Range | Structural | Localizable | Description                                                                                                                              |
| --------------- | ------------------------- | ---------- | ----- | ---------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `system_prompt` | str (multiline)           | empty      | —     | —          | ✓           | Instructions for the classifier. Must include the intents_block tag (the lettered intent list); the letters tag ("A, B, C") is optional. |
| `intents`       | list[ClassifyIntentEntry] | empty list | —     | ✓          | —           | The intents to choose from, 2–26. Each has an id (the label written to the output) and a description the model reads.                    |

### Behavior

| Param              | Type  | Default | Range     | Structural | Localizable | Description                                                                                                                 |
| ------------------ | ----- | ------- | --------- | ---------- | ----------- | --------------------------------------------------------------------------------------------------------------------------- |
| `threshold`        | float | 0.5     | 0.0 – 1.0 | —          | —           | Minimum probability (0–1) the winning intent needs; below it = Not Found.                                                   |
| `margin`           | float | 0.15    | 0.0 – 1.0 | —          | —           | Minimum lead (0–1) of the winner over the runner-up; less = Not Found.                                                      |
| `temperature`      | float | 0.0     | 0.0 – 8.0 | —          | —           | Sharpens (below 1) or flattens (above 1) the probabilities before Threshold and Margin. 0 = unchanged.                      |
| `not_found_intent` | str   | empty   | —         | —          | —           | Intent id that means "none of these": if it wins, take Not Found. Also the output on Not Found. May be #uncertain or empty. |

### Output

| Param           | Type         | Default               | Range | Structural | Localizable | Description                                                                                                    |
| --------------- | ------------ | --------------------- | ----- | ---------- | ----------- | -------------------------------------------------------------------------------------------------------------- |
| `output_name`   | str          | empty                 | —     | ✓          | —           | Name later nodes use to read the result: the winning intent id, or Not Found Intent. Empty = this node's name. |
| `notify_client` | NotifyClient | NotifyClient.Disabled | —     | —          | —           | When to send the game an intent_llm_classified event: Disabled, OnFound (Found exit only), or Always.          |

### Advanced

| Param        | Type             | Default | Range | Structural | Localizable | Description                                                                                                             |
| ------------ | ---------------- | ------- | ----- | ---------- | ----------- | ----------------------------------------------------------------------------------------------------------------------- |
| `context`    | ContextOverrides | unset   | —     | ✓          | —           | Advanced memory and batching settings for this node's model (KV cache type, GPU offload, batch sizes). Unset = inherit. |
| `adaptation` | ModelAdaptation  | unset   | —     | ✓          | —           | LoRA adapters applied to this node's model. Empty = the base model.                                                     |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit        | Param field      | Description                                                                                  |
| ----------- | ---------------- | -------------------------------------------------------------------------------------------- |
| `found`     | `found_exit`     | Node to run when an intent wins clearly (passes Threshold and Margin). Empty = end the turn. |
| `not_found` | `not_found_exit` | Node to run when no intent wins clearly. Empty = end the turn.                               |

LoRA adapters

`adaptation.loras` runs this node's model with one or more fine-tuned LoRA adapters, each with a scale. See [Use a LoRA Adapter](https://docs.tryllengine.com/v0.10/how-to/use-a-lora-adapter.html) for where to put the file and how to tune the scale.

## Exit routes

`temperature` (0 = 1.0, mutable) rescales the scored letter probabilities (`p ← p^(1/T)`) before threshold/margin. Diagnostics emit `parameters.temperature` and, when T ≠ 1, `output.classification.probs_raw`.

| Route       | Fires when                                                                                                             |
| ----------- | ---------------------------------------------------------------------------------------------------------------------- |
| `found`     | Valid message, top probability passes `threshold` and `margin`, and winning intent is not the sink.                    |
| `not_found` | Empty message, failed thresholds, internal scoring error, or winning intent equals `not_found_intent` (`sink_intent`). |

`notify_client`: `OnFound` emits `event_type="intent_llm_classified"` on the found path; `Always` also emits on every `not_found` path. KV: `intent`, `top_prob`, `second_prob`, `not_found_reason`, plus one `<label>.prob` per `intents_ids` entry. Subscribe on the agent (`set_on_intent_llm_classified`, `SetOnIntentLlmClassified`, `TryllAgentComponent.IntentLlmClassified`, `UTryllAgentComponent::OnIntentLlmClassified`). Session events are the all-agents form `(agentId, classification)`; the payload has no agent id. A typed subscriber swallows generic `OnNodeEvent` in the same scope only.

## Diagnostics

When `enable_diagnostics = true`, this node contributes to `debug_info.nodes[].diagnostics`. See [Turn Diagnostics JSON](https://docs.tryllengine.com/v0.10/reference/turn-diagnostics-json.html) for the envelope, the shared `input.prompt[]` shape, and the `engine` counters.

The shared [turn inspector](https://docs.tryllengine.com/v0.10/reference/turn-inspector.html#intent) renders this payload as the **Intent** block. Dialog Lab can also assert the accepted label with an **Intent is** / **Intent not found** check — see [Compare dialog variants](https://docs.tryllengine.com/v0.10/how-to/compare-dialog-variants.html). Raw JSON remains the escape hatch for every field below.

| Key                                      | Meaning                                                                                                                                                                                | Absent when                                                                             |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `parameters.model_name`                  | The model that actually ran, after fallback.                                                                                                                                           | The name could not be resolved.                                                         |
| `parameters.input`                       | The configured input slot name.                                                                                                                                                        | Using the `user_message` default.                                                       |
| `parameters.output_name`                 | The slot this node writes (the node name when `output_name` is empty).                                                                                                                 | —                                                                                       |
| `parameters.threshold`                   | Minimum `top_prob` required to accept a label.                                                                                                                                         | —                                                                                       |
| `parameters.margin`                      | Minimum `top_prob − second_prob` required.                                                                                                                                             | —                                                                                       |
| `parameters.notify_client`               | `Disabled`, `OnFound` or `Always`.                                                                                                                                                     | —                                                                                       |
| `parameters.history_turns`               | How many prior turns were included in the classification prompt.                                                                                                                       | —                                                                                       |
| `parameters.intents_ids[]`               | The label ids **in letter order** — element 0 is `A`, element 1 is `B`, and so on.                                                                                                     | —                                                                                       |
| `input.query`                            | The `input`-resolved text classified. Same string as `output.classification.query` (kept for one release).                                                                             | The resolved text was empty.                                                            |
| `input.prompt[]`                         | The classification prompt as the model received it.                                                                                                                                    | —                                                                                       |
| `output.classification.query`            | The `input`-resolved text classified. Prefer `input.query`.                                                                                                                            | The resolved text was empty.                                                            |
| `output.classification.probs[]`          | Full first-token probability vector. **`probs[i]` is the probability of `parameters.intents_ids[i]`** (letter A = 0). Equal length on `found`; `score_size_mismatch` when they differ. | Scoring did not run.                                                                    |
| `output.classification.top_prob`         | Probability of the winning letter.                                                                                                                                                     | Scoring did not run.                                                                    |
| `output.classification.second_prob`      | Probability of the runner-up letter.                                                                                                                                                   | Scoring did not run.                                                                    |
| `output.classification.margin_result`    | `top_prob − second_prob`, to compare against `parameters.margin`.                                                                                                                      | Scoring did not run.                                                                    |
| `output.classification.predicted_letter` | The winning letter (`A`, `B`, …).                                                                                                                                                      | Nothing was predicted.                                                                  |
| `output.classification.intent`           | The intent label attached.                                                                                                                                                             | The node exited `not_found`.                                                            |
| `output.classification.rejected_intent`  | The label the scorer picked, when routing rejected it.                                                                                                                                 | `found`, or there was no pick (`empty_message`, `score_failed`, `score_size_mismatch`). |
| `output.classification.slot_value`       | The value written to `slot.<output_name>` this turn.                                                                                                                                   | The node has not run.                                                                   |
| `output.classification.not_found_reason` | `empty_message`, `score_size_mismatch`, `score_failed`, `below_threshold`, `below_margin` or `sink_intent`.                                                                            | The node found a match.                                                                 |
| `engine.*`                               | Scheduler counters for the scoring pass — `last_job.kind` is `score`, not `decode`.                                                                                                    | Server config `include_engine_diagnostics` is off.                                      |

To read a `not_found`: `not_found_reason` names the gate that rejected it, and `top_prob` / `margin_result` against `parameters.threshold` / `parameters.margin` show by how much. Pair `probs[]` with `intents_ids[]` positionally to see the full ranking.

## Example

```
from tryll_client.graph import (
    GraphDescription, ClassifyIntentLLMParams, ClassifyIntentEntry, NotifyClient,
)

CLASSIFIER_SYSTEM = (
    "You are an intent classifier for a murder-mystery interrogation.\n"
    "The inspector (user) questions Mr Rixman, the butler (assistant).\n"
    "Pick the single best label for the inspector's LATEST message only.\n\n"
    "Labels:\n{{intents_block}}\n\n"
    "Respond with a single letter. No other text."
)

graph = (
    GraphDescription()
    .add_node("classify", ClassifyIntentLLMParams(
        model_name="Llama 3.2 1B Instruct (Q4_K_M)",
        intents=[
            ClassifyIntentEntry(id="confess_margaret_saw",
                prompt="inspector confronts butler with Mrs Hollis witness statement"),
            ClassifyIntentEntry(id="when_saw_body",
                prompt="inspector asks when butler first saw the body"),
            ClassifyIntentEntry(id="other", prompt="anything else"),
        ],
        system_prompt=CLASSIFIER_SYSTEM,   # required — must reference intents_block
        history_turns=4,
        threshold=0.5,
        notify_client=NotifyClient.OnFound,
        found_exit="pick_instruction",
        not_found_exit="retrieve_world",
    ))
    # ... rest of graph nodes
    .set_start_node("classify")
    .set_default_model_name("My Local Model")
)
```

```
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

constexpr const char* kClassifierSystem =
    "You are an intent classifier for a murder-mystery interrogation.\n"
    "The inspector (user) questions Mr Rixman, the butler (assistant).\n"
    "Pick the single best label for the inspector's LATEST message only.\n\n"
    "Labels:\n{{intents_block}}\n\n"
    "Respond with a single letter. No other text.";

ClassifyIntentLLMParamsT cp;
cp.model_name    = "Llama 3.2 1B Instruct (Q4_K_M)";
auto addIntent = [&](const char* id, const char* prompt)
{
    auto e = std::make_unique<ClassifyIntentEntryT>();
    e->id = id;
    e->prompt = prompt;
    cp.intents.push_back(std::move(e));
};
addIntent("confess_margaret_saw",
    "inspector confronts butler with Mrs Hollis witness statement");
addIntent("when_saw_body", "inspector asks when butler first saw the body");
addIntent("other", "anything else");
cp.system_prompt = kClassifierSystem;  // required — must reference intents_block
cp.history_turns  = 4;
cp.threshold      = 0.5f;
cp.notify_client  = Tryll::NotifyClient_OnFound;
cp.found_exit     = "pick_instruction";
cp.not_found_exit = "retrieve_world";

GraphDescription graph;
graph.AddClassifyIntentLLM("classify", std::move(cp));
// ... rest of graph nodes
```

## When to use this node

|                    | Classify Intent (embedding)         | Classify Intent (LLM)                |
| ------------------ | ----------------------------------- | ------------------------------------ |
| **Requires KB**    | Yes — labelled examples needed      | No                                   |
| **Inference cost** | Embedding only (fast)               | One LM forward pass                  |
| **Intent set**     | Fixed at storage creation           | Defined inline as an `intents` array |
| **Best for**       | Large, pre-labelled KB; low latency | Small label sets; no pre-built KB    |

See `tryll_test_chat` agent `butler2` (`IntentLLMWorkflow`) for a full graph alongside retrieval-based `butler`.

## Related

- [How-to: Use a LoRA adapter](https://docs.tryllengine.com/v0.10/how-to/use-a-lora-adapter.html)
- [How-to: Design an intent-classifier prompt](https://docs.tryllengine.com/v0.10/how-to/design-an-intent-classifier-prompt.html) — the template contract, writing and ordering labels, tuning `threshold`/`margin`
- [How-to: Classify intent without a knowledge base](https://docs.tryllengine.com/v0.10/how-to/classify-intent-without-a-knowledge-base.html) — the end-to-end graph in all four clients
- [How-to: Build an immersion guard](https://docs.tryllengine.com/v0.10/how-to/build-an-immersion-guard.html) — two measured classifiers in front of an NPC
- [Classify Intent](https://docs.tryllengine.com/v0.10/reference/nodes/classify-intent.html) — embedding retrieval alternative
- [Intent to Instruction](https://docs.tryllengine.com/v0.10/reference/nodes/intent-to-instruction.html)
- Research: `docs/research/workflow/intent-classification-slm-logprob.md` (engine design)
- Prompt guide: `docs/research/workflow/intent-classification-logprob-prompt-best-practices.md`
