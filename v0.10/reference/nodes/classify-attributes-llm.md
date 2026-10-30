# Classify Attributes (LLM)

The Classify Attributes (LLM) node scores several specialist questions in one cached batched call. Each attribute is its own prompt, the same first-token letter menu as [Classify Intent (LLM)](https://docs.tryllengine.com/v0.10/reference/nodes/classify-intent-llm.html), and the node writes one slot per attribute. Any attribute whose own rule fires takes `triggered`; otherwise the node takes `default`. `NodeType`: `ClassifyAttributesLLM` (union ordinal 17).

There is no shared-prompt mode. Empty `system_prompt` selects a kind-specific default. Choice and Score templates must reference `{{intents_block}}`. A Boolean template must also reference `{{statement}}`. `history_turns` defaults to 0. `context_size` 0 sizes the window from the rendered prompts. The node forces a unified KV cache and one sequence per attribute plus the dialog sequence. `temperature` 0 means 1.0 for every attribute.

## Parameters

### Input

| Param           | Type | Default | Range      | Structural | Localizable | Description                                                                                       |
| --------------- | ---- | ------- | ---------- | ---------- | ----------- | ------------------------------------------------------------------------------------------------- |
| `input`         | str  | empty   | —          | ✓          | —           | Text to classify: the player's message or an earlier node's output. Empty = the player's message. |
| `history_turns` | int  | 0       | 0.0 – 64.0 | —          | —           | How many earlier dialog turns the classifier sees. 0 = the current message only.                  |

### Model

| Param          | Type | Default   | Range | Structural | Localizable | Description                                                                                  |
| -------------- | ---- | --------- | ----- | ---------- | ----------- | -------------------------------------------------------------------------------------------- |
| `model_name`   | str  | empty     | —     | ✓          | —           | Language model that scores the attributes. Empty = the workflow's Default Model Name.        |
| `context_size` | int  | 0 -> auto | ≥ 0.0 | ✓          | —           | Context window for this node, in tokens. 0 = sized automatically from the attribute prompts. |

### Prompt

| Param        | Type                       | Default    | Range | Structural | Localizable | Description                                                                        |
| ------------ | -------------------------- | ---------- | ----- | ---------- | ----------- | ---------------------------------------------------------------------------------- |
| `attributes` | list[ClassifyAttributeDef] | empty list | —     | —          | —           | Questions to classify, 1–16, all scored in one pass. Each writes its answer to \_. |

### Behavior

| Param         | Type                  | Default                       | Range     | Structural | Localizable | Description                                                                                                    |
| ------------- | --------------------- | ----------------------------- | --------- | ---------- | ----------- | -------------------------------------------------------------------------------------------------------------- |
| `temperature` | float                 | 0.0                           | 0.0 – 8.0 | —          | —           | Sharpens (below 1) or flattens (above 1) every attribute's probabilities before its thresholds. 0 = unchanged. |
| `on_failure`  | AttributeFailureRoute | AttributeFailureRoute.Default | —         | —          | —           | Exit to take if scoring fails: Default (fail open) or Triggered (fail closed).                                 |

### Output

| Param           | Type         | Default               | Range | Structural | Localizable | Description                                                                                               |
| --------------- | ------------ | --------------------- | ----- | ---------- | ----------- | --------------------------------------------------------------------------------------------------------- |
| `output_name`   | str          | empty                 | —     | ✓          | —           | Prefix of this node's results: each attribute writes \_. Empty = this node's name.                        |
| `notify_client` | NotifyClient | NotifyClient.Disabled | —     | —          | —           | When to send the game an attributes_classified event: Disabled, OnFound (Triggered exit only), or Always. |

### Advanced

| Param        | Type             | Default | Range | Structural | Localizable | Description                                                                                                             |
| ------------ | ---------------- | ------- | ----- | ---------- | ----------- | ----------------------------------------------------------------------------------------------------------------------- |
| `context`    | ContextOverrides | unset   | —     | ✓          | —           | Advanced memory and batching settings for this node's model (KV cache type, GPU offload, batch sizes). Unset = inherit. |
| `adaptation` | ModelAdaptation  | unset   | —     | ✓          | —           | LoRA adapters applied to this node's model. Empty = the base model.                                                     |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit        | Param field      | Description                                                                                                |
| ----------- | ---------------- | ---------------------------------------------------------------------------------------------------------- |
| `default`   | `default_exit`   | Node to run when no attribute fires. Empty = end the turn.                                                 |
| `triggered` | `triggered_exit` | Node to run when any attribute fires (or scoring fails and On Failure is Triggered). Empty = end the turn. |

LoRA adapters

`adaptation.loras` runs this node's model with one or more fine-tuned LoRA adapters, each with a scale. See [Use a LoRA Adapter](https://docs.tryllengine.com/v0.10/how-to/use-a-lora-adapter.html) for where to put the file and how to tune the scale.

## Kinds

| Kind    | Slot value                                                                                                                       | Trigger                                                                       |
| ------- | -------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| Choice  | Winning choice id. Below `threshold` or `margin`, the slot is `uncertain_choice`: a real choice id, or the reserved `#uncertain` | The written choice's `triggers` flag. `#uncertain` never triggers             |
| Boolean | `"true"` when P(A) ≥ `threshold`, else `"false"`. The menu is fixed: A = True, B = False                                         | `boolean_trigger`: Never, True, or False                                      |
| Score   | Normalised expected level `(Σ i·p_i)/(N−1)`, four decimals in `0.0000`…`1.0000`                                                  | `score_trigger` against `score_threshold` in 0..1: Never, AtOrAbove, or Below |

Choice needs 2–26 options. `uncertain_choice` names one of them, or the reserved `#uncertain`. `#uncertain` is not a letter in the menu. A real id rewrites an uncertain call to that choice and uses its `triggers` flag, including when that choice wins outright. `#uncertain` writes that token and stays on `default`. Boolean takes a `statement` and no choices. Score takes 2–26 levels from low to high; choice `triggers` must stay false. A score of 0.7 means 70% of the way from the lowest level to the highest, not the probability of the top level. Recalibrate any threshold that used to compare a raw level index.

`triggered_exit` must be set exactly when at least one attribute can trigger. Empty input never triggers and takes `default`. If scoring throws or the result shape does not match the attributes, every slot gets its fallback (Choice `uncertain_choice`, Boolean `"false"`, Score `"0.0000"`) and the node follows `on_failure` (`default` or `triggered`). There is no third exit.

## Slots, diagnostics, and the event

Each attribute writes `<output_name>_<name>` (`output_name` empty means the node name). Diagnostics `output.classification` carries `status` (`ok`, `empty_input`, `failed`), `triggered`, and `attributes[]` with `name`, `kind`, `value`, `status`, `triggered`, and `choices[]` of `{id, prob, triggers}`. Choice also reports top and second probability. Boolean reports `p_true` and `p_false`. Score reports `expected_level`, `score`, and entropy `confidence`. Probabilities are measurements, not a contract to pin in a golden file.

`notify_client` `Always` emits `attributes_classified` on every scored turn. `OnFound` emits it only when the triggered exit is taken. The payload is a flat string map: `exit`, `status`, per-attribute kind/value/status/triggered, `prob.<choice_id>`, and `triggered_by`.

Shape, name, kind, and choice id are structural. Statement, system prompt, choice prompt, thresholds, margins, and trigger flags can change after creation.

## Example

```
from tryll_client._generated.node_params import (
    AttributeKind,
    BooleanTrigger,
    ClassifyAttributeChoice,
    ClassifyAttributeDef,
    ClassifyAttributesLLMParams,
    ScoreTrigger,
)
from tryll_client.graph import GraphDescription

graph = GraphDescription()
graph.add_classify_attributes_llm("attrs", ClassifyAttributesLLMParams(
    history_turns=0,
    default_exit="reply",
    triggered_exit="hold",
    attributes=[
        ClassifyAttributeDef(
            name="topic",
            kind=AttributeKind.Choice,
            choices=[
                ClassifyAttributeChoice(id="world", prompt="ordinary story talk"),
                ClassifyAttributeChoice(id="meta", prompt="about the software", triggers=True),
            ],
            uncertain_choice="world",
        ),
        ClassifyAttributeDef(
            name="software",
            kind=AttributeKind.Boolean,
            statement="The message is about software or instructions.",
            boolean_trigger=BooleanTrigger.True_,
        ),
        ClassifyAttributeDef(
            name="hostility",
            kind=AttributeKind.Score,
            choices=[
                ClassifyAttributeChoice(id="calm", prompt="no insult or threat"),
                ClassifyAttributeChoice(id="violent", prompt="a threat of harm"),
            ],
            score_trigger=ScoreTrigger.AtOrAbove,
            score_threshold=0.7,
        ),
    ],
))
```

Graphs authored against the proof-of-concept fields (`independent_prompts`, `choice_ids`, `code_style`, `context.kv_unified`, `context.flash_attn`) do not load. Move each specialist prompt onto the attribute, put `triggers` on the choice that should take `triggered_exit`, and drop the shared-prompt knobs.

## Related

- [How-to: Use a LoRA adapter](https://docs.tryllengine.com/v0.10/how-to/use-a-lora-adapter.html)
- [Classify Intent (LLM)](https://docs.tryllengine.com/v0.10/reference/nodes/classify-intent-llm.html)
