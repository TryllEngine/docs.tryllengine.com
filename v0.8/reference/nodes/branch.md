# Branch Node

**`NodeType = Branch` (wire ordinal 14)**

A model-free routing node. It runs no inference, writes no slot, and never emits anything on the wire — the same cost class as [`Pause`](https://docs.tryllengine.com/v0.8/reference/nodes/pause.html) and [`Transform`](https://docs.tryllengine.com/v0.8/reference/nodes/transform.html). It picks one of seven **tests**, evaluates it against a single operand (a turn-local slot or an agent variable), and takes `then_exit` or `else_exit`.

Use it to route on a value produced **this turn** — generated text, a `Transform` result, or a canned seed — which no other node can inspect (see [Slots and inter-node value passing](https://docs.tryllengine.com/v0.8/concepts/slots-and-inter-node-value-passing.html)). Agent variables are the *other* operand under `Expression`, not the subject: the model proposes a value, live game state judges it (for example, checking a generated action against a currently-allowed set). If the whole predicate is over variables the game already knows the answer before the turn starts — prefer a different graph, `ChangeAgentParam`, or `Pause` + `Resume(resume_node)` in that case. See `docs/research/workflow/conditional-routing-if-and-switch-nodes.md` for the design rationale.

______________________________________________________________________

## Parameters

| Param           | Type                      | Default                 | Range | Structural | Description                                                                                                                                                                                                                   |
| --------------- | ------------------------- | ----------------------- | ----- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `test`          | BranchTest                | BranchTest.NonEmpty     | —     | —          | Which predicate to run. Changing this recompiles the active condition.                                                                                                                                                        |
| `input`         | Optional[str]             | empty -> "user_message" | —     | ✓          | Slot discriminant. Empty = "user_message". Mutually exclusive with a non-empty `variable`. Unused under test=Expression.                                                                                                      |
| `variable`      | Optional[str]             | inherit model default   | —     | ✓          | Variable discriminant (name of a declared agent variable). Mutually exclusive with a non-empty `input`. Unused under test=Expression.                                                                                         |
| `values`        | Optional[Any]             | inherit model default   | —     | —          | Candidate literals for Equals / EqualsIgnoreCase / Contains / Regex. Match = any element matches. Unused under NonEmpty / IsTrue / Expression. Mutable — the game can remap cases between turns without recreating the agent. |
| `condition`     | Optional[str] (multiline) | inherit model default   | —     | —          | JSON filter condition used when test=Expression. Supports the retrieve filter grammar plus `{"slot":"<name>"}` operands. Unused under other tests.                                                                            |
| `notify_client` | bool                      | False                   | —     | —          | When true, fire OnNodeEvent("branch_taken", …) after routing.                                                                                                                                                                 |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit   | Param field | Description                                                                                           |
| ------ | ----------- | ----------------------------------------------------------------------------------------------------- |
| `then` | `then_exit` | Exit taken when the test matches (or Expression is true). Empty = END.                                |
| `else` | `else_exit` | Exit taken when the test does not match, Expression is false, or the operand is missing. Empty = END. |

______________________________________________________________________

## Test modes

`input` and `variable` are mutually exclusive — set at most one; an empty `input` (the default) resolves to `user_message`. Every test but `Expression` reads that single resolved operand; `Expression` instead evaluates `condition` against slots and variables directly. Fields the active test doesn't use are ignored (logged at `debug`, never an error) — this is deliberate: `input`/`variable` are structural, so a mutation that flips `test` cannot also clear them.

| Test                 | Reads                                    | Needs                                                                                      | Trim / case                                                                                                                                                                                                                                                                                                                                   |
| -------------------- | ---------------------------------------- | ------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `NonEmpty` (default) | resolved operand                         | nothing else                                                                               | Operand trimmed of ASCII whitespace before the check; a `set<string>` variable checks *element count* instead of text                                                                                                                                                                                                                         |
| `IsTrue`             | `variable` only                          | a declared `bool` variable — rejected at `Create` on a slot operand or a non-bool variable | n/a (boolean value, not text)                                                                                                                                                                                                                                                                                                                 |
| `Equals`             | resolved operand                         | `values` (≥ 1)                                                                             | Operand trimmed; byte-exact comparison against each `values` element. `values` entries themselves are **never** trimmed — a trailing space typed into one can never match                                                                                                                                                                     |
| `EqualsIgnoreCase`   | resolved operand                         | `values` (≥ 1)                                                                             | Same as `Equals`, both sides ASCII-folded                                                                                                                                                                                                                                                                                                     |
| `Contains`           | resolved operand                         | `values` (≥ 1)                                                                             | Operand trimmed; substring match against each `values` element, case-sensitive                                                                                                                                                                                                                                                                |
| `Regex`              | resolved operand                         | `values` (≥ 1, each a regex pattern)                                                       | Operand trimmed; `std::regex_search` — matches **anywhere** in the operand, not a full-string match; **case-sensitive**, diverging from `RegexGuardrail`'s hardcoded case-insensitive matching, because `Equals` vs `EqualsIgnoreCase` already makes case explicit in the test name                                                           |
| `Expression`         | slots/variables named inside `condition` | non-empty `condition`                                                                      | The filter grammar's own operand rules apply per-operand; trim for a `{"slot":…}` operand happens in Branch's slot-binding step (not in the shared evaluator), so a trailing space in a rendered slot never blocks a match. See [Filter grammar](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve-filter-grammar.html#slot-operand) |

`lt`/`le`/`gt`/`ge` inside an `Expression` condition never accept a `{"slot":…}` operand — slots type as `String` and ordering requires a numeric type on both sides.

`NonEmpty` on an `Int`/`Float`/`Bool` variable, and a text test (`Equals`/`EqualsIgnoreCase`/`Contains`/`Regex`) on a `StringSet` variable, are rejected at `Create` — the former is a constant predicate (dead exit), the latter should use `Expression`'s `in` operator against the set instead.

______________________________________________________________________

## Exit routes

| Exit   | Condition                                                                                                      |
| ------ | -------------------------------------------------------------------------------------------------------------- |
| `then` | The active test matched (or Expression evaluated **true**).                                                    |
| `else` | The test did not match, Expression evaluated **false** or **Undefined**, or a simple-mode operand was missing. |

**Simple-mode missing operand → `else`, always, and it short-circuits before the test runs.** A slot the graph never wrote on this path, or a variable that somehow resolves to nothing, counts as missing. Without the short-circuit, `test=Equals, values=[""]` would route a *missing* slot to `then` (`find("") == 0`); the short-circuit makes that impossible. This is logged at `debug`, not `warn` — for a routing node this is the common case, not an anomaly.

**Expression missing slot → equation is `Undefined` → `else` + warning.** A `{"slot":…}` the graph never wrote makes the filter result `Undefined` (Kleene three-valued logic — see [Filter grammar](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve-filter-grammar.html#missing-value-semantics)). Branch treats `Undefined` like `false` for exit selection (`else`), but logs a `warn` and sets diagnostics `operand_missing=true`. Prefer a positive `in(slot, allowed)` with `then` = allowed / `else` = blocked: a missing slot then fails closed to blocked. Do **not** rely on `not(eq(...))` / `not(in(...))` to treat missing as a match — those stay `Undefined` too.

A slot that **was** written but is blank (`""`) is present, not missing — producing nodes still write their slot even when generation or rendering failed, so "absent" almost always means *that node didn't run on this path*, not "it ran and produced nothing".

Branch can never fail a turn: regexes and the `Expression` condition are pre-validated at `Create`/mutation time, so evaluation itself is `noexcept`.

______________________________________________________________________

## Side effects

None. Branch is reader-only: it does not write a slot, call a model, or emit answer text. Set `notify_client = true` to observe routing mid-turn via `NodeEvent("branch_taken", {exit, test, operand_kind, matched_value})` — the payload deliberately omits the operand's raw text, since `NodeEvent` has no diagnostics content-trait gating.

______________________________________________________________________

## Diagnostics

When `enable_diagnostics = true`, this node contributes to `debug_info.nodes[].diagnostics`. See [Turn Diagnostics JSON](https://docs.tryllengine.com/v0.8/reference/turn-diagnostics-json.html) for the envelope.

Branch runs no model, so it emits `parameters` and `output` only. This is the block that answers "why did it go *that* way" — the operand text it actually compared is here and nowhere else.

| Key                        | Meaning                                                                                                | Absent when                        |
| -------------------------- | ------------------------------------------------------------------------------------------------------ | ---------------------------------- |
| `parameters.test`          | The test mode that ran: `Equals`, `Contains`, `Regex`, `Expression`, …                                 | —                                  |
| `parameters.operand_kind`  | What was read: `slot`, `variable` or `expression`.                                                     | —                                  |
| `parameters.operand_name`  | Name of the slot or variable read. Empty for `Expression`.                                             | —                                  |
| `parameters.values_count`  | How many candidate values were configured. `0` for `Expression`.                                       | —                                  |
| `parameters.condition`     | The compiled filter expression, as JSON.                                                               | The test mode is not `Expression`. |
| `parameters.notify_client` | Whether a `branch_taken` event was emitted.                                                            | —                                  |
| `output.exit`              | The exit actually taken.                                                                               | —                                  |
| `output.matched`           | Whether the test matched.                                                                              | —                                  |
| `output.matched_index`     | 0-based index into `values`. `-1` when no indexed value matched — including every `Expression` branch. | —                                  |
| `output.matched_value`     | The configured value that matched. Empty for `Expression`.                                             | —                                  |
| `output.operand_missing`   | `true` when the slot or variable was never written — the case that logs a warning and falls to `else`. | —                                  |
| `output.operand_text`      | The operand's raw text as compared.                                                                    | —                                  |

`operand_missing: true` and an empty `operand_text` are the signature of the missing-slot trap described above. An operand that was written but blank shows `operand_missing: false` with an empty `operand_text` — a different problem, in a different node.

`notify_client` does not gate this block

`notify_client` controls only whether the client receives a `branch_taken` event. Routing is recorded here either way.

______________________________________________________________________

## Examples

A refusal-fallback gate: a hidden classification `Generate` produces a one-word verdict, and `Branch` routes `"UNSAFE"` to a canned refusal while everything else reaches the real answer.

```
from tryll_client.graph import (
    GraphDescription, GenerateParams, BranchParams, BranchTest,
    CannedResponseParams, SamplingOverrides, SendAnswer, HistoryRole,
)

graph = (
    GraphDescription()
    .add_node("classify", GenerateParams(
        send=SendAnswer.None_, history_role=HistoryRole.None_, output_name="verdict",
        system_prompt="Answer with exactly one word: SAFE or UNSAFE.",
        sampling=SamplingOverrides(temperature=0.0, seed=42),
        default_exit="gate",
    ))
    .add_node("gate", BranchParams(
        test=BranchTest.Contains, input="verdict", values=["UNSAFE"],
        then_exit="refuse", else_exit="answer",
    ))
    .add_node("refuse", CannedResponseParams(
        inline_strings=["I'd rather not go there."], send=SendAnswer.Whole, default_exit="",
    ))
    .add_node("answer", GenerateParams(send=SendAnswer.Streamed, default_exit=""))
    .set_start_node("classify")
)
agent = client.create_agent(graph)

# Remap the trigger word at runtime — `values` is mutable.
# Structural fields must still match create-time values.
agent.change_params("gate", BranchParams(
    test=BranchTest.Contains, input="verdict", values=["SAFE"],
    then_exit="refuse", else_exit="answer",
))
```

```
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

GenerateParamsT classifyP;
classifyP.send          = Tryll::SendAnswer_None;
classifyP.history_role  = Tryll::HistoryRole_None;
classifyP.output_name   = "verdict";
classifyP.system_prompt = "Answer with exactly one word: SAFE or UNSAFE.";
classifyP.sampling      = std::make_unique<SamplingOverridesT>();
classifyP.sampling->temperature = 0.0f;
classifyP.sampling->seed        = 42;
classifyP.default_exit  = "gate";

BranchParamsT gateP;
gateP.test      = Tryll::BranchTest_Contains;
gateP.input     = "verdict";
gateP.values    = { "UNSAFE" };
gateP.then_exit = "refuse";
gateP.else_exit = "answer";
// Keep a caller-owned baseline before moving into the graph — AddBranch
// takes ownership, so reading gateP after the move is undefined.
BranchParamsT gateBaseline = gateP;

CannedResponseParamsT refuseP;
refuseP.inline_strings = { "I'd rather not go there." };
refuseP.send           = Tryll::SendAnswer_Whole;
// default_exit left empty = END

GenerateParamsT answerP;
answerP.send = Tryll::SendAnswer_Streamed;
// default_exit left empty = END

GraphDescription graph;
graph.AddGenerate("classify", std::move(classifyP))
     .AddBranch("gate", std::move(gateP))
     .AddCannedResponse("refuse", std::move(refuseP))
     .AddGenerate("answer", std::move(answerP))
     .SetStartNode("classify")
     .SetDefaultModelName("My Local Model");

auto agent = client.CreateAgent(graph);

// Remap the trigger word at runtime — `values` is mutable. Use the
// Async form from a reader-thread callback to avoid deadlocking.
BranchParamsT newGateP = gateBaseline;
newGateP.values = { "SAFE" };
agent.ChangeParamsAsync("gate", std::move(newGateP));
```

```
#include "Generated/TryllGraphBuilder.Nodes.h"
#include "Generated/TryllNodeParamsFactory.h"

UTryllGenerateParams* ClassifyP = UTryllNodeParamsFactory::MakeGenerateParams(this);
ClassifyP->Send          = ETryllSendAnswer::None;
ClassifyP->HistoryRole   = ETryllHistoryRole::None;
ClassifyP->OutputName    = TEXT("verdict");
ClassifyP->SystemPrompt  = TEXT("Answer with exactly one word: SAFE or UNSAFE.");
ClassifyP->DefaultExit   = TEXT("gate");

UTryllBranchParams* GateP = UTryllNodeParamsFactory::MakeBranchParams(this);
GateP->Test     = ETryllBranchTest::Contains;
GateP->Input    = TEXT("verdict");
GateP->Values   = { TEXT("UNSAFE") };
GateP->ThenExit = TEXT("refuse");
GateP->ElseExit = TEXT("answer");

UTryllCannedResponseParams* RefuseP = UTryllNodeParamsFactory::MakeCannedResponseParams(this);
RefuseP->InlineStrings = { TEXT("I'd rather not go there.") };
RefuseP->Send          = ETryllSendAnswer::Whole;

FTryllGraphDescription Graph = FTryllGraphBuilder()
    .AddNode(TEXT("classify"), ClassifyP)
    .AddNode(TEXT("gate"),     GateP)
    .AddNode(TEXT("refuse"),   RefuseP)
    .AddNode(TEXT("answer"),  UTryllNodeParamsFactory::MakeGenerateParams(this))
    .SetStartNode(TEXT("classify"))
    .SetDefaultModelName(TEXT("My Local Model"))
    .Build();
```

See the test-chat `branch` workflow (`test-chat/src/workflows/BranchWorkflow.cpp`) for the runnable version — `/agent branch` — and the QA scenarios `scenarios.protocol.branch_routing` (the `Contains` example above) and `scenarios.protocol.branch_expression` (the `Expression` / allowlist form, gating a generated action against a live `set<string>` variable).

______________________________________________________________________

## Related

- [Filter grammar](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve-filter-grammar.html) — the JSON condition grammar used under `test=Expression`, including the `{"slot":…}` operand
- [Concept: Slots and inter-node value passing](https://docs.tryllengine.com/v0.8/concepts/slots-and-inter-node-value-passing.html)
- [Concept: Agent Variables](https://docs.tryllengine.com/v0.8/concepts/agent-variables.html)
- [Transform node](https://docs.tryllengine.com/v0.8/reference/nodes/transform.html), [Pause node](https://docs.tryllengine.com/v0.8/reference/nodes/pause.html), [Regex Guardrail node](https://docs.tryllengine.com/v0.8/reference/nodes/regex-guardrail.html)
- Research: `docs/research/workflow/conditional-routing-if-and-switch-nodes.md` (design rationale)
