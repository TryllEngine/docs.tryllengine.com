# Pause and Resume a Turn

Suspend a turn **between nodes** — react to a tool call, or any graph checkpoint, by mutating node parameters before the turn continues, without waiting for the turn to end and starting a new one.

Prerequisites

- An agent created and ready to receive messages — see [Build a Chat Agent with a Graph](https://docs.tryllengine.com/v0.8/how-to/build-chat-agent-with-graph.html).
- A graph with a pause trigger: either a [`Pause`](https://docs.tryllengine.com/v0.8/reference/nodes/pause.html) node, or a [`ToolCall`](https://docs.tryllengine.com/v0.8/reference/nodes/tool-call.html) node whose `disposition` is one of `Pause`, `PauseAndAcknowledge`, or `AwaitResult`.

______________________________________________________________________

## How it works

Two things can pause a turn:

- **`Pause` node** — a no-op checkpoint. The executor always pauses right after it exits.
- **`ToolCall` with a pausing `disposition`** (`Pause`, `PauseAndAcknowledge`, `AwaitResult`) — pauses only when the node exits `tool_called` (a tool call was detected). No pause on `no_tool_called`, and no pause at all for the non-pausing dispositions (`RouteOnly`, `Acknowledge`, `Notify`, `NotifyAndAcknowledge`). See [ToolCall node reference](https://docs.tryllengine.com/v0.8/reference/nodes/tool-call.html) for the full `ToolCallDisposition` matrix.

While paused:

- The turn stays **open** — `SendMessage` still returns `3004 AgentBusy`.
- [`ChangeAgentParam`](https://docs.tryllengine.com/v0.8/how-to/change-agent-parameters.html) becomes **allowed** — the one thing a normal in-flight turn forbids. This is the reason to pause: inspect a tool call, then mutate a downstream node's params before it runs.
- There is **no timeout**. `Cancel` is the only way to abort a stuck pause — design your UI so the player always has a way to cancel (or your own server-side watchdog if this agent is unattended).

Resume it with `resume_node`:

| `resume_node`   | Effect                                                                                                                                                                           |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Empty (default) | Continue via the paused node's wired exit route.                                                                                                                                 |
| A node name     | Jump directly to that node, skipping the wired exit. Fails with `3005 UnknownNode` if it doesn't exist in the graph — the turn **stays paused**, so a bad name is safe to retry. |

When the pause follows a `ToolCall` with `disposition = AwaitResult`, also pass a complete `tool_results` batch (one `{call_id, result}` per pending call). Echo the `call_id` from the `tool_call` event (`ToolCallNotificationWithId` / `set_on_tool_call_with_id` / …). An invalid batch returns `3016 InvalidToolResults` and leaves the turn paused for retry. An empty/omitted `tool_results` on a plain `Pause` node or a `Pause`/`PauseAndAcknowledge` `ToolCall` is a plain continue; the same empty batch on a pending `AwaitResult` batch is itself rejected with `3016` (the server does not silently treat "no results" as "no tool was called").

| Client         | Plain resume                                                        | Tool-result resume                                                                                                                                       |
| -------------- | ------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Unity / Unreal | `Resume()`                                                          | `ResumeWithToolResult(s)` or `RegisterTool` (auto-resume; retains the batch across a failed attempt — see `HasPausedToolBatch` / `RetryPausedToolBatch`) |
| C++            | `Resume` / `ResumeAsync`                                            | `ResumeWithToolResult(s)` / `*Async`                                                                                                                     |
| Python         | `resume()` (never from a reader-thread callback) / `resume_async()` | `resume_async(tool_results=[ToolResult(...), ...])` from a callback, or blocking `resume(tool_results=...)` off the reader thread                        |

______________________________________________________________________

## Examples

```
using Tryll.Client;

var agentComp = GetComponent<TryllAgentComponent>();

TryllClient.Instance.Paused += (agentId, nodeName, pendingExit) =>
{
    Debug.Log($"[paused] at {nodeName} -> {pendingExit}");

    // Mutate a downstream node while paused — allowed here, would be
    // AgentBusy during a normal in-flight turn.
    var p = (TryllGenerateParams)agentComp.GetNodeParamsBaseline("answer");
    p.SystemPrompt = "Recent tool result: porch light is now on.";
    agentComp.ChangeParams("answer", p, err =>
    {
        if (!err.IsOk) { Debug.LogError(err.Message); return; }
        agentComp.Resume(); // continue via the pending exit
    });
    // Completions also fire OnParamChanged / OnResumed UnityEvents.
};
```

```
auto* subsystem = GetGameInstance()->GetSubsystem<UTryllSubsystem>();
subsystem->OnPaused.AddDynamic(this, &ThisClass::HandlePaused);

void AThisClass::HandlePaused(int64 AgentId, const FString& NodeName, const FString& PendingExit)
{
    UTryllNodeParamsBase* Base = AgentComponent->GetNodeParamsBaseline(TEXT("answer"));
    UTryllGenerateParams* P = Cast<UTryllGenerateParams>(
        UTryllNodeParamsFactory::CloneParams(Base, this));
    P->SystemPrompt = TEXT("Recent tool result: porch light is now on.");
    AgentComponent->ChangeParams(TEXT("answer"), P);
    AgentComponent->Resume(); // continue via the pending exit
}
```

1. Bind **On Paused** on `UTryllSubsystem` (or `OnPaused` on the `TryllAgentComponent`'s owning subsystem).
1. Clone + mutate the target node's params, as in [Change Agent Parameters at Runtime](https://docs.tryllengine.com/v0.8/how-to/change-agent-parameters.html).
1. Call **Resume** on the `TryllAgentComponent`, optionally with a node name to jump to.

```
agent.SetOnPaused(
    [&agent](std::string_view nodeName, std::string_view pendingExit)
    {
        // Fires on the reader thread — mutate via the async form, don't
        // block waiting for the Ack this same thread must read next.
        GenerateParamsT p = answerBaseline;
        p.system_prompt = "Recent tool result: porch light is now on.";
        agent.ChangeParamsAsync("answer", std::move(p));
        agent.ResumeAsync(); // empty resume_node = continue via pendingExit
    });
```

```
from concurrent.futures import ThreadPoolExecutor
from copy import deepcopy

_work = ThreadPoolExecutor(max_workers=1)

def _apply_and_resume(node_name: str) -> None:
    # Runs off the reader thread: change_params has no async form, so
    # it is safe here but would deadlock the reader if called directly
    # from on_paused below.
    p = deepcopy(answer_baseline)
    p.system_prompt = "Recent tool result: porch light is now on."
    agent.change_params(node_name, p)
    agent.resume()  # empty resume_node = continue via pending_exit

def on_paused(node_name: str, pending_exit: str) -> None:
    print(f"[paused] at {node_name} -> {pending_exit}")
    # Fires on the reader thread — never call blocking client methods
    # (resume(), change_params()) here directly; either hand off to
    # another thread (as above) or use resume_async() for the resume
    # step alone if no param mutation is needed.
    _work.submit(_apply_and_resume, "answer")

agent.set_on_paused(on_paused)
```

`agent.resume(resume_node="", tool_results=(), timeout=30.0)` blocks waiting for `Ack` and raises `TryllError` on failure (`3012 AgentNotPaused`, `3005 UnknownNode`, `3016 InvalidToolResults`) — safe only when called from a thread that is not the client's reader thread (e.g. your own application thread, not a `set_on_paused` / `set_on_tool_call_with_id` callback). From those callbacks, use `agent.resume_async(...)` (returns a `Future` instead of blocking) when no other blocking call is needed, or hand the whole sequence off to a worker thread as above.

______________________________________________________________________

## Jumping instead of continuing

Pass a node name to route the game's own decision, bypassing the graph's wired exit entirely — useful when the pause is a generic checkpoint (a `Pause` node) and the *game*, not the graph author, decides what happens next:

```
// Player declined the trade — skip "confirm" and go straight to "decline".
agentComp.Resume("decline");
```

```
agent.Resume("decline");
```

```
agent.resume("decline")
```

______________________________________________________________________

## Combining with a `ToolCall` loop

For a single-round "call → result → answer" graph, set `disposition = AwaitResult`, pause after the call, resume with results, then let a downstream `Generate` speak:

```
flowchart LR
    tc["ToolCall<br>detect<br>(disposition=AwaitResult)"]
    gen["Generate<br>answer"]
    tc -. "tool_called (paused)<br>ResumeWithToolResult(s)" .-> gen
    tc -- "no_tool_called" --> gen
```

This single-pass shape (one call → one result → one downstream answer) is the only pattern currently validated end-to-end (see `scenarios.tools.tool_result_v1`). **Repeated result-bearing jumps back into the same `ToolCall` node (an agentic multi-round loop) are not supported**: correlation IDs (`c%08x`) restart at `c00000000` on every `ToolCallNode::Execute()`, so a second pass in the same interaction can collide with the first, and there is no regression test for it. Model a bounded number of rounds as separate `ToolCall` nodes in the graph instead of looping through one, until unique per-interaction exchange IDs and a two-pause regression test exist.

Side-effect tools that do not need a client result can use the default `disposition = Acknowledge` (or `RouteOnly`) and skip `tool_results` entirely — see [Define and handle tool calls](https://docs.tryllengine.com/v0.8/how-to/define-and-handle-tool-calls.html).

______________________________________________________________________

## Error codes

| Code   | Name                 | Cause                                                                                                                                 |
| ------ | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| `3012` | `AgentNotPaused`     | `Resume` was called but the agent is idle, or actively running (not paused).                                                          |
| `3005` | `UnknownNode`        | The `resume_node` you passed does not name a node in the graph. The turn stays paused.                                                |
| `3016` | `InvalidToolResults` | `tool_results` is incomplete, has unknown/duplicate ids, was sent on a non-tool pause, or exceeds size limits. The turn stays paused. |

______________________________________________________________________

## Related

- [Reference: Pause node](https://docs.tryllengine.com/v0.8/reference/nodes/pause.html)
- [Reference: ToolCall node](https://docs.tryllengine.com/v0.8/reference/nodes/tool-call.html)
- [Define and Handle Tool Calls](https://docs.tryllengine.com/v0.8/how-to/define-and-handle-tool-calls.html)
- [Change Agent Parameters at Runtime](https://docs.tryllengine.com/v0.8/how-to/change-agent-parameters.html)
- [Reference: Wire Protocol — Pausing and resuming a turn](https://docs.tryllengine.com/v0.8/reference/wire-protocol.html#pausing-and-resuming-a-turn)
