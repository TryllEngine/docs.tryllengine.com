# Generate

The Generate node runs [language-model](https://docs.tryllengine.com/v0.10/reference/glossary.html#language-model) inference. It is the workhorse of every Tryll [workflow](https://docs.tryllengine.com/v0.10/reference/glossary.html#workflow): it takes the current [dialog](https://docs.tryllengine.com/v0.10/reference/glossary.html#dialog), builds a [prompt](https://docs.tryllengine.com/v0.10/reference/glossary.html#prompt) from it that fits the node's context window, and emits the model's response — either as a single chunk or streamed [token-by-token](https://docs.tryllengine.com/v0.10/reference/glossary.html#token).

`NodeType`: `Generate`.

Leave `context_size` at `0` to inherit the model variant's window, then the server `default_n_ctx` (commonly 8192). ClassifyIntentLLM and ToolCall use a 2048-token node default instead. The `context` table is an advanced structural override (`kv_cache_type`, veto-only `offload_kqv`, escape-hatch batch sizes). Do not set `q4_0` KV expecting a VRAM win at a right-sized window — measured savings show up only at large `n_ctx`. `sampling.max_tokens` is fixed at creation: it sizes the prompt budget (`context_size` minus `max_tokens`) and cannot be changed with `ChangeAgentParam`. When it is omitted, the node inherits the model's own `default_sampling.max_tokens`; 758 is the built-in value only when the model sets none. It must be at least 1 and strictly less than the context size.

## Parameters

### Input

| Param   | Type | Default | Range | Structural | Localizable | Description                                                                                             |
| ------- | ---- | ------- | ----- | ---------- | ----------- | ------------------------------------------------------------------------------------------------------- |
| `input` | str  | empty   | —     | ✓          | —           | Text this node answers: the player's message or an earlier node's output. Empty = the player's message. |

### Model

| Param          | Type | Default | Range | Structural | Localizable | Description                                                                                                                       |
| -------------- | ---- | ------- | ----- | ---------- | ----------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `model_name`   | str  | empty   | —     | ✓          | —           | Language model this node runs. Empty = the workflow's Default Model Name.                                                         |
| `context_size` | int  | 0       | ≥ 0.0 | ✓          | —           | Context window for this node, in tokens; must fit the model's trained maximum. 0 = the model's own size, else the server default. |

### Prompt

| Param           | Type            | Default                      | Range | Structural | Localizable | Description                                                                                                                                  |
| --------------- | --------------- | ---------------------------- | ----- | ---------- | ----------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `system_prompt` | str (multiline) | empty                        | —     | —          | ✓           | System message the model reads before the dialog. Empty = none.                                                                              |
| `template`      | str (multiline) | empty                        | —     | —          | ✓           | Mustache text added next to the Input (where: see Placement), for example the slot. tag or retrieved knowledge. Empty = none.                |
| `placement`     | Placement       | Placement.BeforeUserAsSystem | —     | —          | —           | Where Template goes: before or after the Input, as a system or a user message.                                                               |
| `grammar`       | str (multiline) | empty                        | —     | —          | —           | GBNF grammar the answer must follow; needs a `root` rule. Empty = free text. Use it to force a format, such as JSON or a fixed set of lines. |

### Output

| Param                        | Type                  | Default               | Range | Structural | Localizable | Description                                                                                                                   |
| ---------------------------- | --------------------- | --------------------- | ----- | ---------- | ----------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `output_name`                | str                   | empty                 | —     | ✓          | —           | Name later nodes use to read this node's answer (in their Input, or as the slot. tag). Empty = this node's name.              |
| `send`                       | SendAnswer            | SendAnswer.Streamed   | —     | —          | —           | How the answer reaches the game: Streamed (token by token), Whole (once, at the end), or None (kept for later nodes only).    |
| `history_role`               | HistoryRole           | HistoryRole.Assistant | —     | —          | —           | Whether later turns remember this answer as an assistant message. None = forgotten after this turn.                           |
| `substitute_agent_variables` | bool                  | False                 | —     | —          | —           | Replace **NAME** markers in the answer with the values of agent variables that allow output substitution. Not case-sensitive. |
| `output_filter`              | OutputFilterOverrides | unset                 | —     | —          | —           | Removes roleplay artifacts from the answer: "Bob:" prefixes, *actions*, (asides), [notes]. All off by default.                |

### Advanced

| Param        | Type              | Default | Range | Structural | Localizable | Description                                                                                                             |
| ------------ | ----------------- | ------- | ----- | ---------- | ----------- | ----------------------------------------------------------------------------------------------------------------------- |
| `sampling`   | SamplingOverrides | unset   | —     | —          | —           | Overrides for the model's sampling settings (temperature, top-p, max tokens…). Unset values keep the model's own.       |
| `context`    | ContextOverrides  | unset   | —     | ✓          | —           | Advanced memory and batching settings for this node's model (KV cache type, GPU offload, batch sizes). Unset = inherit. |
| `adaptation` | ModelAdaptation   | unset   | —     | ✓          | —           | LoRA adapters applied to this node's model. Empty = the base model.                                                     |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit      | Param field    | Description                                                      |
| --------- | -------------- | ---------------------------------------------------------------- |
| `default` | `default_exit` | Node to run after the answer is generated. Empty = end the turn. |

LoRA adapters

`adaptation.loras` runs this node's model with one or more fine-tuned LoRA adapters, each with a scale. See [Use a LoRA Adapter](https://docs.tryllengine.com/v0.10/how-to/use-a-lora-adapter.html) for where to put the file and how to tune the scale.

## Exit routes

| Route     | Param field    | Fires when                                                                                                                 |
| --------- | -------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `default` | `default_exit` | Always — Generate never branches. Set `default_exit` to the next node name, or leave it empty to terminate the turn (END). |

## Side effects

- Keeps the generated answer in the current [turn](https://docs.tryllengine.com/v0.10/reference/glossary.html#interaction) as the assistant's reply unless `history_role = None`, in which case later turns do not remember it.
- Emits `AnswerText` frames to the client according to `send`: token deltas for `Streamed`, one frame for `Whole`, nothing for `None` (a hidden step whose text only later nodes read).
- Updates this node's own [KV cache](https://docs.tryllengine.com/v0.10/reference/glossary.html#context-kv) (one per node, per agent).

## Constrained output (GBNF grammar)

Set the optional `grammar` param to a [GBNF](https://github.com/ggml-org/llama.cpp/blob/master/grammars/README.md) grammar and the model's output is constrained to that grammar at **every decode step** — malformed output becomes literally unsampleable. This is ideal for closed-set command parsing (`<action> <object>`), dialogue-choice enums, emotion tags, or any output consumed by game code rather than read by a human.

```
root   ::= action " " object
action ::= "grab" | "pull" | "cut" | "activate"
object ::= "wire" | "enemy" | "lever" | "door"
```

With this grammar every reply is one of the 4×4 = 16 legal strings — no parse failures, no "Sure! I think you meant…" preamble to strip.

What to keep in mind:

- **The grammar must contain a `root` rule.** An invalid grammar fails **fast**: agent creation is rejected (`GraphCompilationFailed`), and a bad `ChangeAgentParam` mutation is rejected with `InvalidParamValue` (3007).
- **Grammar guarantees syntax, never semantics.** On garbled input the model still picks the highest-probability *allowed* token — it can emit a valid-but-wrong command. Grammar removes junk *options*; it does not know what the player meant. Gate low-confidence commands upstream (e.g. with [ClassifyIntentLLM](https://docs.tryllengine.com/v0.10/reference/nodes/classify-intent-llm.html)) when that matters.
- **Constrain the wire, not the thinking.** On a reasoning model, a grammar that forbids `<think>` tokens breaks or degrades generation. Turn thinking off (per-variant `disable_thinking`) when constraining command output.
- **Mutable.** `grammar` is rebuilt per turn, so a client can flip the node between a strict "command turn" (grammar set) and free chat (grammar empty) via `ChangeAgentParam`.
- **Termination.** A grammar with no path to end-of-generation (e.g. `root ::= "a" root`) never lets the model stop; only `max_tokens` bounds it.

## Diagnostics

When `enable_diagnostics = true`, this node contributes to `debug_info.nodes[].diagnostics`. See [Turn Diagnostics JSON](https://docs.tryllengine.com/v0.10/reference/turn-diagnostics-json.html) for the envelope, the shared `input.prompt[]` shape, and the `engine` counters.

| Key                                                                                                                                  | Meaning                                                                                                                                                 | Absent when                                        |
| ------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| `parameters.model_name`                                                                                                              | The model that actually ran, after the `model_name` / `default_model_name` fallback.                                                                    | The name could not be resolved.                    |
| `parameters.temperature`, `top_p`, `top_k`, `max_tokens`, `min_p`, `seed`, `repeat_penalty`, `presence_penalty`, `frequency_penalty` | The resolved sampling set.                                                                                                                              | —                                                  |
| `parameters.send`                                                                                                                    | `None`, `Whole` or `Streamed`.                                                                                                                          | —                                                  |
| `parameters.history_role`                                                                                                            | How the answer replays into later turns.                                                                                                                | —                                                  |
| `parameters.substitute_agent_variables`                                                                                              | `true`.                                                                                                                                                 | Output substitution is off.                        |
| `parameters.output_filter.strip_speaker_prefix`                                                                                      | `true`. One key per enabled rule; the four rule keys are `strip_speaker_prefix`, `strip_asterisk_spans`, `strip_paren_spans` and `strip_bracket_spans`. | That rule is off.                                  |
| `parameters.output_filter.strip_asterisk_spans`                                                                                      | `true`.                                                                                                                                                 | That rule is off.                                  |
| `parameters.output_filter.strip_paren_spans`                                                                                         | `true`.                                                                                                                                                 | That rule is off.                                  |
| `parameters.output_filter.strip_bracket_spans`                                                                                       | `true`.                                                                                                                                                 | That rule is off.                                  |
| `parameters.output_filter.max_prefix_bytes`, `parameters.output_filter.max_span_bytes`                                               | The scan limits the filter ran with.                                                                                                                    | No filter rule is enabled.                         |
| `parameters.input`                                                                                                                   | The configured input slot name.                                                                                                                         | Using the `user_message` default.                  |
| `parameters.output_name`                                                                                                             | The configured output slot name.                                                                                                                        | Defaulting to the node name.                       |
| `parameters.template`                                                                                                                | The knowledge template, **truncated to 120 characters** with a trailing `...`.                                                                          | No template is set.                                |
| `parameters.placement`                                                                                                               | Where the rendered template was placed.                                                                                                                 | No template is set.                                |
| `parameters.grammar_constrained`                                                                                                     | `true` when a GBNF grammar constrained sampling.                                                                                                        | No grammar is set.                                 |
| `input.prompt[]`                                                                                                                     | The prompt as the model received it — history replay, template and trimming already applied.                                                            | —                                                  |
| `output.text`                                                                                                                        | The text this node produced.                                                                                                                            | The node produced no output slot.                  |
| `engine.*`                                                                                                                           | Grammar state and scheduler counters.                                                                                                                   | Server config `include_engine_diagnostics` is off. |

Two node-level arrays sit **outside** `diagnostics` and are visible nowhere else in the turn inspector:

- `variable_replacements[]` — every `__VARIABLE__` marker replaced, with byte offsets in both the raw and the transformed stream. See [Substitute agent variables in output](https://docs.tryllengine.com/v0.10/how-to/substitute-agent-variables-in-output.html).
- `output_filter_removals[]` — every span a filter rule deleted. See [Filter LLM output artifacts](https://docs.tryllengine.com/v0.10/how-to/filter-llm-output-artifacts.html).

Both cap at 256 entries and carry a `*_truncated` companion flag.

## Minimum working example

```
from tryll_client.graph import GraphDescription, GenerateParams

graph = (
    GraphDescription()
    .add_node("answer", GenerateParams(
        system_prompt="You are a terse assistant.",
        default_exit="",   # empty = END
    ))
    .set_start_node("answer")
    .set_default_model_name("My Local Model")
)

agent = client.create_agent(graph)
```

```
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

GenerateParamsT gp;
gp.system_prompt = "You are a terse assistant.";
// gp.default_exit = ""; // empty = END (the default)

GraphDescription graph;
graph.AddGenerate("answer", std::move(gp))
     .SetStartNode("answer")
     .SetDefaultModelName("My Local Model");

auto agent = client.CreateAgent(graph);
```

```
using Tryll.Client;

var graph = new TryllGraphBuilder()
    .AddGenerate("answer", new TryllGenerateParams
    {
        SystemPrompt = "You are a terse assistant.",
        // DefaultExit = ""; // empty = END (the default)
    })
    .SetStartNode("answer")
    .SetDefaultModelName("My Local Model")
    .Build();
```

```
#include "Generated/TryllGraphBuilder.Nodes.h"
#include "Generated/TryllNodeParamsFactory.h"

UTryllGenerateParams* P = UTryllNodeParamsFactory::MakeGenerateParams(this);
P->bOverrideSystemPrompt = true;
P->SystemPrompt = TEXT("You are a terse assistant.");
// P->DefaultExit = TEXT(""); // empty = END (the default)

FTryllGraphDescription Graph = FTryllGraphBuilder()
    .AddNode(TEXT("answer"), P)
    .SetStartNode(TEXT("answer"))
    .SetDefaultModelName(TEXT("My Local Model"))
    .Build();
```

Or author the same node inside a `UTryllWorkflowAsset` in the Content Browser and assign it to `UTryllAgentComponent`.

## Client bindings

- **C++:** `GraphDescription::AddGenerate(name, GenerateParamsT)` — `GraphDescription.h`
- **Python:** `GraphDescription.add_node(name, GenerateParams(...))` — `tryll_client.graph`
- **Unity:** `TryllGraphBuilder.AddGenerate(name, new TryllGenerateParams{...})` — `Runtime/Generated/TryllGraphBuilder.Nodes.cs`
- **Unreal:** `AddGenerateNode(builder, name, UTryllGenerateParams*)` — `Generated/TryllGraphBuilder.Nodes.h`

## Related

- [How-to: Use a LoRA adapter](https://docs.tryllengine.com/v0.10/how-to/use-a-lora-adapter.html)
- [Concept: Models and inference engines](https://docs.tryllengine.com/v0.10/concepts/models-and-inference-engines.html)
- [Concept: Dialog History and the Context Window](https://docs.tryllengine.com/v0.10/concepts/dialog-history-and-the-context-window.html)
- [Concept: Constrained output (GBNF)](https://docs.tryllengine.com/v0.10/concepts/constrained-output.html)
- [How to stream answers to a UI](https://docs.tryllengine.com/v0.10/how-to/stream-answers-to-ui.html)
- [How to constrain output with a grammar](https://docs.tryllengine.com/v0.10/how-to/constrain-output-with-a-grammar.html)
- [How to filter LLM output artifacts](https://docs.tryllengine.com/v0.10/how-to/filter-llm-output-artifacts.html)
- [How to substitute agent variables in LLM output](https://docs.tryllengine.com/v0.10/how-to/substitute-agent-variables-in-output.html)
- [Agent Parameters](https://docs.tryllengine.com/v0.10/reference/agent-parameters.html#default-model)
- [Glossary](https://docs.tryllengine.com/v0.10/reference/glossary.html#language-model)
