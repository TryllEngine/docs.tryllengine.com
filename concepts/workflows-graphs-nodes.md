# Workflows, Graphs, and Nodes

A Tryll [agent](https://docs.tryllengine.com/reference/glossary.html#agent) is not a prompt. It is a *graph* of small, single-purpose steps that you wire together. Each user turn re-walks the graph; each step decides where to go next. This page explains why Tryll is built this way and how the pieces fit.

## The building blocks

A [graph](https://docs.tryllengine.com/reference/glossary.html#graph) has two things:

- A set of **[nodes](https://docs.tryllengine.com/reference/nodes/index.html)**, each with a name, a [node type](https://docs.tryllengine.com/reference/glossary.html#node), and a set of **typed params** that include the node's configuration *and* its exit wiring. Each exit is a string field on the params object (e.g. `default_exit`, `triggered_exit`); an empty value means *route to END*. The full set of 14 node types is: [`Generate`](https://docs.tryllengine.com/reference/nodes/generate.html), [`GenerateAndSpeak`](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html), [`Speak`](https://docs.tryllengine.com/reference/nodes/speak.html), [`Retrieve`](https://docs.tryllengine.com/reference/nodes/retrieve.html), [`ToolCall`](https://docs.tryllengine.com/reference/nodes/tool-call.html), [`CannedResponse`](https://docs.tryllengine.com/reference/nodes/canned-response.html), [`RegexGuardrail`](https://docs.tryllengine.com/reference/nodes/regex-guardrail.html), [`Instruction`](https://docs.tryllengine.com/reference/nodes/instruction.html), [`IntentToInstruction`](https://docs.tryllengine.com/reference/nodes/intent-to-instruction.html), [`ClassifyIntent`](https://docs.tryllengine.com/reference/nodes/classify-intent.html), [`ClassifyIntentLLM`](https://docs.tryllengine.com/reference/nodes/classify-intent-llm.html), [`Transform`](https://docs.tryllengine.com/reference/nodes/transform.html), [`Pause`](https://docs.tryllengine.com/reference/nodes/pause.html), and [`Branch`](https://docs.tryllengine.com/reference/nodes/branch.html). See [Workflow Nodes](https://docs.tryllengine.com/reference/nodes/index.html) for the full catalog.
- A **start node** — the first node to run on every user turn.

Graphs are authored once, sent to the server with [`CreateAgentRequest`](https://docs.tryllengine.com/reference/agent-parameters.html), frozen at agent creation, and then re-walked for every turn the agent lives.

Here is the simplest useful graph — a guardrail in front of a generator, with a scripted refusal on the rejected path:

```
flowchart LR
    guard["RegexGuardrail<br>guard"]
    refuse["CannedResponse<br>refuse"]
    gen["Generate<br>answer"]
    guard -- "triggered" --> refuse
    guard -- "not_triggered" --> gen
    refuse -- "default" --> END
    gen -- "default" --> END
```

The same graph, in each client:

```
var graph = new TryllGraphBuilder()
    .AddRegexGuardrail("guard", new TryllRegexGuardrailParams
    {
        StringStorage    = "jailbreak_patterns",
        TriggeredExit    = "refuse",  // → refuse node
        NotTriggeredExit = "answer",  // → answer node
    })
    .AddCannedResponse("refuse", new TryllCannedResponseParams
    {
        StringStorage = "refusal_lines",
        // DefaultExit is "" (END) by default
    })
    .AddGenerate("answer", new TryllGenerateParams())
    .SetStartNode("guard")
    .SetDefaultModelName("My Local Model")
    .Build();
```

```
auto* GuardParams  = NewObject<UTryllRegexGuardrailParams>();
GuardParams->StringStorage    = TEXT("jailbreak_patterns");
GuardParams->TriggeredExit    = TEXT("refuse");
GuardParams->NotTriggeredExit = TEXT("answer");

auto* RefuseParams = NewObject<UTryllCannedResponseParams>();
RefuseParams->StringStorage = TEXT("refusal_lines");
// RefuseParams->DefaultExit defaults to "" (END)

FTryllGraphDescription Graph = FTryllGraphBuilder()
    .AddNode(TEXT("guard"),  GuardParams)
    .AddNode(TEXT("refuse"), RefuseParams)
    .AddNode(TEXT("answer"), NewObject<UTryllGenerateParams>())
    .SetStartNode(TEXT("guard"))
    .SetDefaultModelName(TEXT("My Local Model"))
    .Build();
```

```
namespace TC = Tryll::Client;

TC::RegexGuardrailParams guardParams;
guardParams.string_storage     = "jailbreak_patterns";
guardParams.triggered_exit     = "refuse";
guardParams.not_triggered_exit = "answer";

TC::CannedResponseParams refuseParams;
refuseParams.string_storage = "refusal_lines";
// refuseParams.default_exit defaults to "" (END)

TC::GraphDescription graph;
graph.AddNode("guard",  guardParams)
     .AddNode("refuse", refuseParams)
     .AddNode("answer", TC::GenerateParams{})
     .SetStartNode("guard")
     .SetDefaultModelName("My Local Model");
```

```
from tryll_client import GraphDescription
from tryll_client._generated.nodes import (
    RegexGuardrailParams, CannedResponseParams, GenerateParams,
)

guard_params = RegexGuardrailParams(
    string_storage     = "jailbreak_patterns",
    triggered_exit     = "refuse",
    not_triggered_exit = "answer",
)
refuse_params = CannedResponseParams(
    string_storage = "refusal_lines",
    # default_exit defaults to "" (END)
)

graph = (
    GraphDescription()
    .add_node("guard",  guard_params)
    .add_node("refuse", refuse_params)
    .add_node("answer", GenerateParams())
    .set_start_node("guard")
    .set_default_model_name("My Local Model")
)
```

## Exit fields: how nodes "talk" to each other

Nodes do not call each other. They return a named **exit**, and the agent follows the matching edge. Every node type declares its possible exits as **typed string fields** on its params object — see the table in [Workflow Nodes](https://docs.tryllengine.com/reference/nodes/index.html).

For example:

| Node type        | Exit fields on params                     |
| ---------------- | ----------------------------------------- |
| `Generate`       | `default_exit`                            |
| `Retrieve`       | `found_exit`, `not_found_exit`            |
| `ToolCall`       | `tool_called_exit`, `no_tool_called_exit` |
| `CannedResponse` | `default_exit`                            |
| `RegexGuardrail` | `triggered_exit`, `not_triggered_exit`    |

An **empty string** means *route to END*; this is the field's default value. A **non-empty string** must name another node in the same graph — the server rejects graphs where an exit field names a node that doesn't exist ([`3008 InvalidExitTarget`](https://docs.tryllengine.com/reference/error-codes.html#3xxx-agent)).

Because exits are fields on the params object, exits that terminate the turn (route to END) can simply be left at their default — you only need to set the fields that route somewhere other than END.

## The per-turn loop

When a `SendMessage` arrives, the agent:

1. Appends a new [interaction](https://docs.tryllengine.com/reference/glossary.html#interaction) to its dialog, starting with the user's message.
1. Walks the graph from `start_at`, one node at a time. Each node:
1. reads from the current interaction (and anything earlier nodes attached to it),
1. does its work (retrieve from an index, run the model, match a pattern, pick a canned line, …),
1. optionally attaches new components to the interaction, and
1. returns its chosen exit.
1. Follows the route for that exit to the next node.
1. Stops when the route target is `"END"`, when the step budget is exceeded, or when the turn is cancelled.

What each node attaches along the way is what makes the dialog *itself* useful state. A `Retrieve` node attaches retrieved knowledge to the current turn, and a downstream `Generate` node's [projection](https://docs.tryllengine.com/concepts/projection-and-token-budgets.html) picks that knowledge up and folds it into the prompt.

## Passing values between nodes: slots

Producing nodes (`Generate`, `GenerateAndSpeak`, `CannedResponse`, `Transform`, `Instruction`/`IntentToInstruction`) don't just route — they each write their text into a named **slot** on the turn's shared blackboard (the slot name is the node's `output_name`, or its own name if unset). Downstream nodes select which slot they consume with an `input` param (empty = the reserved `user_message` slot), and Mustache templates reference slots as `{{slot.<name>}}`. Two more per-node knobs control delivery: `send` (a `SendAnswer` of `None` / `Whole` / `Streamed`) decides whether/how the slot is emitted to the client this turn, and `history_role` decides whether it replays as history on later turns. This is how you build things like a hidden query-rewrite step (`send=None`) or a multi-sender turn. See [Slots and inter-node value passing](https://docs.tryllengine.com/concepts/slots-and-inter-node-value-passing.html).

## Why a graph instead of a prompt template

Three reasons:

**1. Composition.** "Retrieve, then generate" is different from "check for jailbreak first, then retrieve, then generate". Prompt templates push this branching into strings; a graph makes it executable and inspectable. Every node is typed and [diagnostics-friendly](https://docs.tryllengine.com/reference/error-codes.html).

**2. Cost control.** The graph lets you short-circuit expensive work. A [guardrail](https://docs.tryllengine.com/reference/nodes/regex-guardrail.html) that routes to a [canned response](https://docs.tryllengine.com/reference/nodes/canned-response.html) never touches the model. A [tool-call node](https://docs.tryllengine.com/reference/nodes/tool-call.html) with `mode=detect_only` does zero extra generation on a miss (the default, `call_or_answer`, instead emits the model's plain-text reply).

**3. Reuse.** Graphs are data. You can build them at runtime from C++, Python, or Unreal, ship them in a [`UTryllWorkflowAsset`](https://docs.tryllengine.com/reference/client-api/unreal/cpp/index.html), or construct them dynamically from user configuration. The server does not know or care where the graph came from.

## Turns are serial; graphs are stateless between turns

An agent runs one turn at a time. If a second `SendMessage` arrives while the previous turn is still running, it is rejected with [`3004 AgentBusy`](https://docs.tryllengine.com/reference/error-codes.html#3xxx-agent); the in-flight turn keeps running. Cancellation is explicit (`CancelRequest`, `DestroyAgent`, socket close), not pre-emption.

The graph itself carries no per-turn state. All state lives on the agent's dialog (the accumulated interactions) and in the per-node KV caches. This is why the same graph description can be replayed cleanly every turn — the work of "what has been said" is separate from the work of "what to do next".

## Edges and pitfalls

- **Routing loops.** Nothing stops you from routing node A → node A; the agent has a configurable `max_steps_per_turn` budget (default 64) that breaks infinite loops by aborting the turn with an error.
- **Exits you "know" will never fire still need a value.** Leave them at the default empty string (routes to END). The server validates that no exit field names a missing node, but it does not enforce that every exit field is reachable.
- **Model sharing is *cross-graph*, not cross-node.** Two `Generate` nodes in one graph that use the same model share the underlying weights (good) but have independent KV caches (necessary — they see different prompts). Budget token-count accordingly; see [Projection and Token Budgets](https://docs.tryllengine.com/concepts/projection-and-token-budgets.html).
- **The start node runs every turn.** If your start node is `Generate`, it will generate every turn, even if a later node would have short-circuited. Put guardrails and retrieval *before* the model, not after, if you want them to actually gate work.

## Related

- [Reference: Workflow Nodes](https://docs.tryllengine.com/reference/nodes/index.html)
- [Reference: Agent Parameters](https://docs.tryllengine.com/reference/agent-parameters.html)
- [Concept: Agents and Sessions](https://docs.tryllengine.com/concepts/agents-and-sessions.html)
- [Concept: Projection and Token Budgets](https://docs.tryllengine.com/concepts/projection-and-token-budgets.html)
- [Concept: Tool Calling](https://docs.tryllengine.com/concepts/tool-calling.html)
- [Concept: Slots and inter-node value passing](https://docs.tryllengine.com/concepts/slots-and-inter-node-value-passing.html)
- [How-to: Build a chat agent with a graph](https://docs.tryllengine.com/how-to/build-chat-agent-with-graph.html)
- [How-to: Edit workflows in the Unity graph editor](https://docs.tryllengine.com/how-to/edit-workflows-in-graph-editor.html)
- [How-to: Edit workflows in the Unreal graph editor](https://docs.tryllengine.com/how-to/edit-workflows-in-unreal-graph-editor.html)
- [How-to: Use canned responses and guardrails](https://docs.tryllengine.com/how-to/use-canned-responses-and-guardrails.html)
