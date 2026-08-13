# Pause Node

**`NodeType = Pause` (wire ordinal 12)**

A no-op node. When it exits, the workflow executor suspends the turn **between nodes** — after `PauseNode` exits but before the graph resolves its next node — instead of continuing immediately. The turn stays open (the client still owns it and it still counts as busy for `SendMessage`), but unlike a normal in-flight turn, [`ChangeAgentParam`](https://docs.tryllengine.com/how-to/change-agent-parameters.html) is allowed while paused.

The node itself does **not** call a model and does **not** modify the conversation history. It exists purely to give game logic a checkpoint to react to before the turn continues.

Resume a paused turn with `ResumeAgentRequest`:

- **Plain resume** (`resume_node` empty) — continues via `default_exit`, same as any other exit.
- **Jump resume** (`resume_node` set) — routes to the named node instead, skipping the node's own wiring. Fails with `3005 UnknownNode` if the name doesn't exist in the graph.

See [How to pause and resume a turn](https://docs.tryllengine.com/how-to/pause-and-resume-turns.html) for the full client-side flow, and [`ToolCall`](https://docs.tryllengine.com/reference/nodes/tool-call.html) for the other way to trigger a pause (a pausing `disposition`: `Pause`, `PauseAndAcknowledge`, or `AwaitResult`).

______________________________________________________________________

## Parameters

| Param | Type | Default | Range | Structural | Description |
| ----- | ---- | ------- | ----- | ---------- | ----------- |
|       |      |         |       |            |             |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit      | Param field    | Description                                                                          |
| --------- | -------------- | ------------------------------------------------------------------------------------ |
| `default` | `default_exit` | Default exit target taken on a plain resume (empty resume_node). Empty string = END. |

______________________________________________________________________

## Exit routes

| Exit      | Condition                                                                        |
| --------- | -------------------------------------------------------------------------------- |
| `default` | Taken on a plain resume (empty `resume_node`) or as the target of a jump resume. |

______________________________________________________________________

## Example

```
from tryll_client.graph import GraphDescription, PauseParams, GenerateParams, Placement

graph = (
    GraphDescription()
    .add_node("checkpoint", PauseParams(default_exit="generate"))
    .add_node("generate", GenerateParams(
        template="{{human_message}}",
        placement=Placement.BeforeUserAsSystem,
        default_exit="",   # empty = END
    ))
    .set_start_node("checkpoint")
    .set_default_model_name("Llama 3.2 3B Instruct (Q4_K_M)")
)
agent = client.create_agent(graph)

agent.set_on_paused(lambda node, exit_route: print(f"paused at {node} -> {exit_route}"))
agent.send_message("hello")
# ... game logic runs, may call agent.change_params(...) here ...
agent.resume()  # or agent.resume("generate") to jump directly
```

```
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

PauseParamsT pp;
pp.default_exit = "generate";

GenerateParamsT gp;
gp.template_ = "{{human_message}}";
gp.placement = ::Tryll::Placement_BeforeUserAsSystem;

GraphDescription graph;
graph.AddPause("checkpoint", std::move(pp))
     .AddGenerate("generate", std::move(gp))
     .SetStartNode("checkpoint")
     .SetDefaultModelName("Llama 3.2 3B Instruct (Q4_K_M)");
auto agent = client.CreateAgent(graph);

agent.SetOnPaused([](std::string_view node, std::string_view exitRoute)
{
    // game logic; may call agent.ChangeParams(...) here
});
agent.SendMessage("hello");
agent.Resume(); // or agent.Resume("generate") to jump directly
```

______________________________________________________________________

## Related

- [Tool Call node](https://docs.tryllengine.com/reference/nodes/tool-call.html) — the other pause trigger (a pausing `disposition`)
- [How-to: Pause and resume a turn](https://docs.tryllengine.com/how-to/pause-and-resume-turns.html)
- [How-to: Change Agent Parameters at Runtime](https://docs.tryllengine.com/how-to/change-agent-parameters.html)
- [Wire Protocol Reference](https://docs.tryllengine.com/reference/wire-protocol.html)
