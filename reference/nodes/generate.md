# Generate

The Generate node runs [language-model](https://docs.tryllengine.com/reference/glossary.html#language-model) inference. It is the workhorse of every Tryll [workflow](https://docs.tryllengine.com/reference/glossary.html#workflow): it takes the current [dialog](https://docs.tryllengine.com/reference/glossary.html#dialog), passes it through the agent's [projection](https://docs.tryllengine.com/reference/glossary.html#projection) to build a [prompt](https://docs.tryllengine.com/reference/glossary.html#prompt), and emits the model's response — either as a single chunk or streamed [token-by-token](https://docs.tryllengine.com/reference/glossary.html#token).

`NodeType`: `Generate`.

## Parameters

| Param                        | Type                      | Default                      | Range | Structural | Description                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ---------------------------- | ------------------------- | ---------------------------- | ----- | ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `model_name`                 | Optional[str]             | inherit model default        | —     | ✓          | Model catalog name. Empty = use the agent's default_model_name.                                                                                                                                                                                                                                                                                                                                                                         |
| `context_size`               | int                       | 0                            | ≥ 0.0 | ✓          | KV-cache / context window (n_ctx) for this node in tokens. 0 = fall back to the model variant's context_size, else the server default_n_ctx. Validated against the model's trained maximum at agent creation.                                                                                                                                                                                                                           |
| `system_prompt`              | Optional[str] (multiline) | inherit model default        | —     | —          | Prepended before the user turn during projection.                                                                                                                                                                                                                                                                                                                                                                                       |
| `output_name`                | Optional[str]             | inherit model default        | —     | ✓          | Name this node's output slot is stored under. Empty = the node's own name. Structural: immutable after creation — renaming would re-wire the slot dataflow that is validated once at agent creation.                                                                                                                                                                                                                                    |
| `input`                      | Optional[str]             | inherit model default        | —     | ✓          | Slot name this node consumes as its primary text (the "user turn" of the projected prompt). Empty = "user_message". Structural: immutable after creation — rebinding would re-wire the slot dataflow that is validated once at agent creation.                                                                                                                                                                                          |
| `template`                   | Optional[str] (multiline) | inherit model default        | —     | —          | Mustache template applied to the user-area message at projection time.                                                                                                                                                                                                                                                                                                                                                                  |
| `placement`                  | Placement                 | Placement.BeforeUserAsSystem | —     | —          | Where the rendered template body is placed relative to the resolved `input`.                                                                                                                                                                                                                                                                                                                                                            |
| `send`                       | SendAnswer                | SendAnswer.Streamed          | —     | —          | Controls answer-text delivery to the client. Replaces the old `stream: bool`.                                                                                                                                                                                                                                                                                                                                                           |
| `history_role`               | HistoryRole               | HistoryRole.Assistant        | —     | —          | Controls whether/how this node's slot replays in later turns' projected transcript.                                                                                                                                                                                                                                                                                                                                                     |
| `sampling`                   | Optional[Any]             | inherit model default        | —     | —          | Sparse sampling overrides applied on top of the model-catalog defaults. Sub-table change fires OnSamplingChanged, which re-resolves overrides against the cached model-default sampling.                                                                                                                                                                                                                                                |
| `grammar`                    | Optional[str] (multiline) | inherit model default        | —     | —          | Optional GBNF grammar. When non-empty, output is constrained to this grammar at every decode step (malformed output is unsampleable). Empty = unconstrained. Must contain a `root` rule. Mutable — rebuilt per turn, so a client can flip the node between a strict "command turn" and free chat via ChangeAgentParam. Validated at agent creation (invalid → agent-create failure) and on mutation (invalid → InvalidParamValue 3007). |
| `substitute_agent_variables` | bool                      | False                        | —     | —          | When true, replace `__VARIABLE__` markers in streamed LLM output with the rendered values of agent variables that declared `allow_output_substitution`. Matching is ASCII case-insensitive. Default false. Transformed text is authoritative for the slot, wire answer, history, and any downstream consumers. Appended after default_exit to keep FlatBuffers field ids additive.                                                      |
| `output_filter`              | Optional[Any]             | inherit model default        | —     | —          | Opt-in bounded artifact cleanup applied to streamed LLM output before variable substitution, slot/history write, wire delivery, and any downstream consumers. Missing/null = all flags off (defaults). Appended after substitute_agent_variables for additive field ids.                                                                                                                                                                |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit      | Param field    | Description                                                                                       |
| --------- | -------------- | ------------------------------------------------------------------------------------------------- |
| `default` | `default_exit` | Default exit target — routes here after generation completes. Empty string = END (turn finishes). |

## Exit routes

| Route     | Param field    | Fires when                                                                                                                 |
| --------- | -------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `default` | `default_exit` | Always — Generate never branches. Set `default_exit` to the next node name, or leave it empty to terminate the turn (END). |

## Side effects

- Appends the generated answer to the current [turn](https://docs.tryllengine.com/reference/glossary.html#interaction) as the assistant's reply.
- Emits one or more `AnswerText` frames to the client.
- Updates the per-model [KV cache](https://docs.tryllengine.com/reference/glossary.html#context-kv).

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
- **Grammar guarantees syntax, never semantics.** On garbled input the model still picks the highest-probability *allowed* token — it can emit a valid-but-wrong command. Grammar removes junk *options*; it does not know what the player meant. Gate low-confidence commands upstream (e.g. with [ClassifyIntentLLM](https://docs.tryllengine.com/reference/nodes/classify-intent-llm.html)) when that matters.
- **Constrain the wire, not the thinking.** On a reasoning model, a grammar that forbids `<think>` tokens breaks or degrades generation. Turn thinking off (per-variant `disable_thinking`) when constraining command output.
- **Mutable.** `grammar` is rebuilt per turn, so a client can flip the node between a strict "command turn" (grammar set) and free chat (grammar empty) via `ChangeAgentParam`.
- **Termination.** A grammar with no path to end-of-generation (e.g. `root ::= "a" root`) never lets the model stop; only `max_tokens` bounds it.

## Diagnostics

When the agent has `enable_diagnostics = true`, the node's contribution to `TurnComplete.debug_info` includes the projected prompt and the model name that was actually run (after the `model_name` / `default_model_name` fallback).

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

- [Concept: Models and inference engines](https://docs.tryllengine.com/concepts/models-and-inference-engines.html)
- [Concept: Projection and token budgets](https://docs.tryllengine.com/concepts/projection-and-token-budgets.html)
- [Concept: Constrained output (GBNF)](https://docs.tryllengine.com/concepts/constrained-output.html)
- [How to stream answers to a UI](https://docs.tryllengine.com/how-to/stream-answers-to-ui.html)
- [How to constrain output with a grammar](https://docs.tryllengine.com/how-to/constrain-output-with-a-grammar.html)
- [How to filter LLM output artifacts](https://docs.tryllengine.com/how-to/filter-llm-output-artifacts.html)
- [How to substitute agent variables in LLM output](https://docs.tryllengine.com/how-to/substitute-agent-variables-in-output.html)
- [Agent Parameters](https://docs.tryllengine.com/reference/agent-parameters.html#default-model)
- [Glossary](https://docs.tryllengine.com/reference/glossary.html#language-model)
