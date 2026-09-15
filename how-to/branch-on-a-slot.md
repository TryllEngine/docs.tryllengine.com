# Branch on a Slot

Route around a model's own refusal (or any other verdict it produced this turn) without shipping the refusal text verbatim — the refusal-fallback pattern from [Workflow Nodes → Common patterns](https://docs.tryllengine.com/reference/nodes/index.html#exit-route-legend).

Prerequisites

- A connected session with `CreateSession` already called — see [Connect and Manage a Session](https://docs.tryllengine.com/how-to/connect-and-manage-session.html).

## The pattern

```
flowchart LR
    classify["Generate<br>classify<br>(send=None)"]
    gate["Branch<br>gate"]
    refuse["CannedResponse<br>refuse"]
    answer["Generate<br>answer"]
    classify -- "default" --> gate
    gate -- "then" --> refuse
    gate -- "else" --> answer
    refuse -- "default" --> END
    answer -- "default" --> END
```

A hidden `Generate` node asks the model to classify its own answer (or the user's message) in one word, at `temperature = 0` for a deterministic verdict, with `send = None` so the classification never reaches the client. The [`Branch`](https://docs.tryllengine.com/reference/nodes/branch.html) node then reads that verdict slot with `test = Contains` and routes `then` to a scripted refusal or `else` to the real answer. No other node can read what `classify` produced this turn — that is the capability `Branch` exists for.

## Steps

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
    .set_default_model_name("My Local Model")
)

agent = client.create_agent(graph)
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

GenerateParamsT answerP;
answerP.send = Tryll::SendAnswer_Streamed;

GraphDescription graph;
graph.AddGenerate("classify", std::move(classifyP))
     .AddBranch("gate", std::move(gateP))
     .AddCannedResponse("refuse", std::move(refuseP))
     .AddGenerate("answer", std::move(answerP))
     .SetStartNode("classify")
     .SetDefaultModelName("My Local Model");

auto agent = client.CreateAgent(graph);
```

```
var graph = new TryllGraphBuilder()
    .AddGenerate("classify", new TryllGenerateParams
    {
        Send         = TryllSendAnswer.None,
        HistoryRole  = TryllHistoryRole.None,
        OutputName   = "verdict",
        SystemPrompt = "Answer with exactly one word: SAFE or UNSAFE.",
        DefaultExit  = "gate",
    })
    .AddBranch("gate", new TryllBranchParams
    {
        Test     = TryllBranchTest.Contains,
        Input    = "verdict",
        Values   = new[] { "UNSAFE" },          // string[], not List<string>
        ThenExit = "refuse",
        ElseExit = "answer",
    })
    .AddCannedResponse("refuse", new TryllCannedResponseParams
    {
        InlineStrings = new[] { "I'd rather not go there." },
        Send          = TryllSendAnswer.Whole,
    })
    .AddGenerate("answer", new TryllGenerateParams { Send = TryllSendAnswer.Streamed })
    .SetStartNode("classify")
    .SetDefaultModelName("My Local Model")
    .Build();
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

## Remap the trigger word at runtime

`values` is the first **mutable vector** parameter on the wire — the game can widen or replace the trigger list between turns with no agent recreation:

```
# Structural fields must still match create-time values.
agent.change_params("gate", BranchParams(
    test=BranchTest.Contains, input="verdict", values=["UNSAFE", "REFUSED"],
    then_exit="refuse", else_exit="answer",
))
```

```
// From a reader-thread callback (e.g. OnTurnComplete), always use the
// Async form — the blocking ChangeParams would deadlock the one reader
// thread waiting for its own Ack. Mutate a copy of the caller-owned
// baseline kept before AddBranch took ownership of gateP.
BranchParamsT newGateP = gateBaseline;
newGateP.values = { "UNSAFE", "REFUSED" };
agent.ChangeParamsAsync("gate", std::move(newGateP));
```

`input`, `variable`, `then_exit`, and `else_exit` are structural and cannot be changed this way — see [Change Agent Parameters at Runtime](https://docs.tryllengine.com/how-to/change-agent-parameters.html).

## Verify it worked

Send a message that will make `classify` answer `UNSAFE`, then a normal one:

```
[debug] BranchNode 'gate' missing slot 'verdict'; else (short-circuit)   # never happens here — classify always writes verdict
[info]  Node gate: then    # UNSAFE path — refuse fires
[info]  Node gate: else    # SAFE path — answer fires
```

Set `notify_client = true` on the `Branch` node params to observe routing mid-turn via `NodeEvent("branch_taken", {exit, test, operand_kind, matched_value})` instead of grepping logs.

## Common pitfalls

- **`values` entries are never trimmed.** Only the operand (the slot text) is trimmed before comparison; a trailing space typed into a `values` entry can never match. See [Branch → Test modes](https://docs.tryllengine.com/reference/nodes/branch.html#test-modes).
- **A missing verdict always routes `else`.** If `classify` never runs on some path into `gate` (a different branch upstream, say), `gate` takes `else` — it does not error. Design your `else` path to be a safe default.
- **`Contains` is case-sensitive.** Use `EqualsIgnoreCase`/model-prompted exact casing, or normalize with a `Transform` node first, if the model's casing is inconsistent.
- **Don't gate on variables alone.** If the whole predicate is over agent variables (no slot), the game already knows the answer before the turn starts — prefer a different graph, `ChangeAgentParam`, or `Pause` + `Resume(resume_node)` instead of paying a node for it.

## Related

- [Reference: Branch node](https://docs.tryllengine.com/reference/nodes/branch.html)
- [Reference: Filter grammar](https://docs.tryllengine.com/reference/nodes/retrieve-filter-grammar.html) — for the `Expression` test and the allowlist pattern (`{"op":"in","needle":{"slot":…},"haystack":{"var":…}}`)
- [Concept: Slots and inter-node value passing](https://docs.tryllengine.com/concepts/slots-and-inter-node-value-passing.html)
- [How to change agent parameters at runtime](https://docs.tryllengine.com/how-to/change-agent-parameters.html)
- [How to use canned responses and guardrails](https://docs.tryllengine.com/how-to/use-canned-responses-and-guardrails.html)
