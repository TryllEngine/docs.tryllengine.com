# Pause Node

**`NodeType = Pause` (wire ordinal 12)**

A no-op node. When it exits, the workflow executor suspends the turn **between nodes** — after `PauseNode` exits but before the graph resolves its next node — instead of continuing immediately. The turn stays open (the client still owns it and it still counts as busy for `SendMessage`), but unlike a normal in-flight turn, [`ChangeAgentParam`](https://docs.tryllengine.com/how-to/change-agent-parameters.html) is allowed while paused.

The node itself does **not** call a model and does **not** modify the conversation history. It exists purely to give game logic a checkpoint to react to before the turn continues.

Resume a paused turn with `ResumeAgentRequest`:

- **Plain resume** (`resume_node` empty) — continues via `default_exit`, same as any other exit.
- **Jump resume** (`resume_node` set) — routes to the named node instead, skipping the node's own wiring. Fails with `3005 UnknownNode` if the name doesn't exist in the graph.

See [How to pause and resume a turn](https://docs.tryllengine.com/how-to/pause-and-resume-turns.html) for the full client-side flow, and [`ToolCall`](https://docs.tryllengine.com/reference/nodes/tool-call.html) for the other way to trigger a pause (a pausing `disposition`: `Pause`, `PauseAndAcknowledge`, or `AwaitResult`).

## Timing out a pause

By default (`pause_timeout_ms = 0`) a pause waits forever. That is the right behaviour when your game is guaranteed to answer — and a soft-lock when it isn't, because the agent stays busy and every later `SendMessage` is rejected with `AgentBusy`. The server logs a warning naming the parked node once a pause outlives `workflow.pause_stall_warn_ms` (15 s by default), and your client logs one immediately if nothing it holds can resume the pause at all.

Set `pause_timeout_ms` to give the pause a deadline. What happens on expiry depends on whether you authored `timeout_exit`:

| `timeout_exit` | On expiry                                                                                                              |
| -------------- | ---------------------------------------------------------------------------------------------------------------------- |
| set            | The graph continues from that exit — author a fallback line, or a `Branch` that picks one. The turn ends normally.     |
| empty          | The turn is abandoned with `TurnStatus.PauseTimedOut`. The partial interaction is kept, so the conversation is intact. |

Prefer authoring a `timeout_exit` for anything player-facing: the character says *something* instead of going quiet.

Two things to keep in mind:

- The deadline is **wall-clock on the server**. A minimised or alt-tabbed game stops pumping its main thread, so the deadline can elapse while your game is suspended. Make the timeout branch harmless — "shrug and answer without the tool" — rather than something that only makes sense the instant it fires.
- `pause_timeout_ms` and `timeout_exit` are **structural**: they are fixed when the agent is created and `ChangeAgentParam` rejects them with `3006 ParamNotMutable`. Vary them per agent, not per turn.

______________________________________________________________________

## Parameters

| Param              | Type | Default | Range | Structural | Description                                                                                                                                                                                                                                                                                                                          |
| ------------------ | ---- | ------- | ----- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `pause_timeout_ms` | int  | 0       | ≥ 0.0 | ✓          | Wall-clock budget for this pause, in milliseconds. 0 = infinite: wait for ResumeAgentRequest or CancelRequest however long that takes (the shipped default, so existing graphs are unaffected). A finite budget is the right choice whenever nothing guarantees the client will resume. Structural: resolved once at agent creation. |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit      | Param field    | Description                                                                                                                                                                                                                                            |
| --------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `default` | `default_exit` | Default exit target taken on a plain resume (empty resume_node). Empty string = END.                                                                                                                                                                   |
| `timeout` | `timeout_exit` | Exit taken when pause_timeout_ms elapses. Empty = end the turn with TurnStatus.PauseTimedOut, keeping the partial interaction. Authoring a target lets the graph answer anyway (a canned fallback line, a Branch). Ignored when pause_timeout_ms is 0. |

______________________________________________________________________

## Exit routes

| Exit      | Condition                                                                        |
| --------- | -------------------------------------------------------------------------------- |
| `default` | Taken on a plain resume (empty `resume_node`) or as the target of a jump resume. |

______________________________________________________________________

## Diagnostics

Pause records **no** diagnostics of its own: its node entry in `debug_info.nodes[]` carries `_type`, `name`, `exit_route` and `duration_s`, with no `diagnostics` object at all.

```
{
  "_type": "PauseNode",
  "name": "checkpoint",
  "exit_route": "default",
  "duration_s": 3.5e-06
}
```

`duration_s` is microseconds even for a pause the client held for minutes, because paused time is excluded from every per-node duration. The pause is measured at the **turn** level instead — `paused_duration_s` and `pause_count` in the [envelope](https://docs.tryllengine.com/reference/turn-diagnostics-json.html#envelope) — and a pause that timed out shows up as the turn `status` `pause_timed_out`.

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
