# Classify Intent (LLM)

The Classify Intent (LLM) node maps a user message to a discrete **intent label** using a small language model's **first-token logprob** over a closed set of letter candidates (`A`, `B`, `C`, …). Each letter maps positionally to an entry in `intents_ids`. On success it attaches an `IntentionComponent` — the same routing-only component produced by [Classify Intent](https://docs.tryllengine.com/reference/nodes/classify-intent.html) — so downstream [IntentToInstruction](https://docs.tryllengine.com/reference/nodes/intent-to-instruction.html) wiring is unchanged.

`NodeType`: `ClassifyIntentLLM`.

## How it works

1. Projects the dialog via `ClassifyIntentLLMProjectionStrategy`:
1. Stable **system** message (Mustache template + pre-rendered label list).
1. Up to `history_turns` past **user** / **assistant** pairs for coreference (history turns always replay the `user_message` slot).
1. Current turn's `input`-resolved slot as the **user** message (empty = `user_message`).
1. Prefills the classifier model (`Sync()`); reuses KV-cache prefix across turns when the prompt tail is append-only.
1. Reads logits at the final position and softmaxes over the single-token ids for `A`, `B`, `C`, …
1. If `top_prob ≥ threshold` and `(top − second) ≥ margin`, attaches `IntentionComponent{intents_ids[top]}` and returns `found`.

No text is generated; no sampling.

## Parameters

| Param              | Type                      | Default               | Range      | Structural | Description                                                                                                                                                                                                               |
| ------------------ | ------------------------- | --------------------- | ---------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `model_name`       | Optional[str]             | inherit model default | —          | ✓          | Model catalog name. Structural because the token-ID mapping is built from the model's vocabulary at construction.                                                                                                         |
| `context_size`     | int                       | 0                     | ≥ 0.0      | ✓          | KV-cache / context window (n_ctx) for this node in tokens. 0 = fall back to the model variant's context_size, else the server default_n_ctx. Validated against the model's trained maximum at agent creation.             |
| `intents_ids`      | Optional[str]             | inherit model default | —          | ✓          | Comma-separated intent IDs (e.g. "greet,farewell,other"). Structural because label-to-token-ID mapping is built at construction.                                                                                          |
| `intents_prompt`   | Optional[str]             | inherit model default | —          | ✓          | Comma-separated intent prompt labels / descriptions. Structural because the projection scaffolding and label-token mapping are built at construction.                                                                     |
| `system_prompt`    | Optional[str] (multiline) | inherit model default | —          | —          | Prepended before the user turn during classification projection.                                                                                                                                                          |
| `input`            | Optional[str]             | inherit model default | —          | ✓          | Slot name this node consumes as its primary text for the current turn. Empty = "user_message". Structural: immutable after creation — rebinding would re-wire the slot dataflow that is validated once at agent creation. |
| `history_turns`    | int                       | 2                     | 0.0 – 64.0 | —          | Number of dialog history turns included in the classification context.                                                                                                                                                    |
| `threshold`        | float                     | 0.5                   | 0.0 – 1.0  | —          | First-token probability threshold for the winning label.                                                                                                                                                                  |
| `margin`           | float                     | 0.15                  | 0.0 – 1.0  | —          | Minimum margin between top and second label probabilities.                                                                                                                                                                |
| `notify_client`    | bool                      | False                 | —          | —          | When true, fire OnNodeEvent("intent_classified", …) on the found path.                                                                                                                                                    |
| `not_found_intent` | Optional[str]             | inherit model default | —          | —          | Intent ID returned when no label clears the threshold/margin.                                                                                                                                                             |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit        | Param field      | Description                                                                                 |
| ----------- | ---------------- | ------------------------------------------------------------------------------------------- |
| `found`     | `found_exit`     | Exit taken when classification produced a label above threshold/margin. Empty string = END. |
| `not_found` | `not_found_exit` | Exit taken when no label cleared the threshold/margin. Empty string = END.                  |

## Exit routes

| Route       | Fires when                                                                                                             |
| ----------- | ---------------------------------------------------------------------------------------------------------------------- |
| `found`     | Valid message, top probability passes `threshold` and `margin`, and winning intent is not the sink.                    |
| `not_found` | Empty message, failed thresholds, internal scoring error, or winning intent equals `not_found_intent` (`sink_intent`). |

## Diagnostics

When diagnostics are enabled on the agent, `TurnComplete.debug_info` includes these keys from the classify node:

| Key                | Description                                                                                                                                           |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `input`            | The configured `input` slot name, when set (absent when using the `user_message` default).                                                            |
| `query`            | The `input`-resolved text classified.                                                                                                                 |
| `top_prob`         | Softmax probability of the winning letter.                                                                                                            |
| `second_prob`      | Softmax probability of the runner-up letter.                                                                                                          |
| `margin_result`    | `top_prob − second_prob`.                                                                                                                             |
| `result.intent`    | Attached intent label (empty on `not_found`).                                                                                                         |
| `probs_json`       | Full probability vector over labels.                                                                                                                  |
| `intents_ids_json` | JSON array of `intents_ids` in label order.                                                                                                           |
| `predicted_letter` | Winning letter (`A`, `B`, …).                                                                                                                         |
| `not_found_reason` | Why `not_found` fired: `empty_message`, `score_size_mismatch`, `score_failed`, `below_threshold`, `below_margin`, or `sink_intent`. Empty on `found`. |

## Example

```
from tryll_client.graph import GraphDescription, ClassifyIntentLLMParams

graph = (
    GraphDescription()
    .add_node("classify", ClassifyIntentLLMParams(
        model_name="Llama 3.2 1B Instruct (Q4_K_M)",
        intents_ids="confess_margaret_saw,when_saw_body,other",
        intents_prompt=(
            "inspector confronts butler with Mrs Hollis witness statement,"
            "inspector asks when butler first saw the body,"
            "anything else"
        ),
        history_turns=4,
        threshold=0.5,
        notify_client=True,
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

ClassifyIntentLLMParamsT cp;
cp.model_name     = "Llama 3.2 1B Instruct (Q4_K_M)";
cp.intents_ids    = "confess_margaret_saw,when_saw_body,other";
cp.intents_prompt =
    "inspector confronts butler with Mrs Hollis witness statement,"
    "inspector asks when butler first saw the body,"
    "anything else";
cp.history_turns  = 4;
cp.threshold      = 0.5f;
cp.notify_client  = true;
cp.found_exit     = "pick_instruction";
cp.not_found_exit = "retrieve_world";

GraphDescription graph;
graph.AddClassifyIntentLLM("classify", std::move(cp));
// ... rest of graph nodes
```

## When to use this node

|                    | Classify Intent (embedding)         | Classify Intent (LLM)             |
| ------------------ | ----------------------------------- | --------------------------------- |
| **Requires KB**    | Yes — labelled examples needed      | No                                |
| **Inference cost** | Embedding only (fast)               | One LM forward pass               |
| **Intent set**     | Fixed at storage creation           | Defined inline as strings         |
| **Best for**       | Large, pre-labelled KB; low latency | Small label sets; no pre-built KB |

See `tryll_test_chat` agent `butler2` (`IntentLLMWorkflow`) for a full graph alongside retrieval-based `butler`.

## Related

- [Classify Intent](https://docs.tryllengine.com/reference/nodes/classify-intent.html) — embedding retrieval alternative
- [Intent to Instruction](https://docs.tryllengine.com/reference/nodes/intent-to-instruction.html)
- Research: `docs/research/workflow/intent-classification-slm-logprob.md` (engine design)
- Prompt guide: `docs/research/workflow/intent-classification-logprob-prompt-best-practices.md`
