# Define and Handle Tool Calls

Declare a tool the model can "call", ship it with a [`ToolCall` node](https://docs.tryllengine.com/reference/nodes/tool-call.html), receive `tool_call` events on the client, and either return a model-visible result on resume (`AwaitResult`) or let the node's disposition close the exchange (`Acknowledge` / `RouteOnly` / …).

Prerequisites

- A session connected, with a session created.
- A language model whose **chat template supports tool calling** (Qwen 3.x, Granite 4.x, Llama-3.x, …). Tool prompting/parsing is driven by the model's own template — there is no format to pick, and `CreateAgent` rejects a model whose template can't render tools. See the [Tool Calling concept](https://docs.tryllengine.com/concepts/tool-calling.html).

## The pattern

```
flowchart LR
    tc["ToolCall<br>detect"]
    gen["Generate<br>answer"]
    tc -- "tool_called" --> gen
    tc -- "no_tool_called" --> gen
    gen -- "default" --> END
```

- **`disposition = AwaitResult`** — notify, pause after the call, client runs the tool, resumes with `tool_results`, then `Generate` answers with the complete call/result pair in prompt.
- **`Acknowledge` (default) / `RouteOnly` / …** — no client result needed; the server closes history itself. Use a notifying disposition (`Notify`, `NotifyAndAcknowledge`, …) when the game must react to the side effect.

## Step 1 — declare the tools

Tools are declared per-node with typed tool definitions:

```
var setLight = new TryllToolDefinition {
    Name        = "set_light",
    Description = "Turn a named light on or off.",
    Parameters  = new List<TryllToolParamDefinition> {
        new() { Name = "name", Type = "string",  Description = "Human-readable name of the light, e.g. 'porch'." },
        new() { Name = "on",   Type = "boolean", Description = "true to turn on, false to turn off." },
    },
};
```

Author `FTryllToolDefinition` entries inside your `UTryllWorkflowAsset` (Content Browser data asset) alongside the graph. Or build them at runtime in C++ and assign to the `UTryllAgentComponent`'s graph.

```
namespace TC = Tryll::Client;

TC::ToolDef setLight{
    "set_light",
    "Turn a named light on or off.",
    {
        {"name", "string",  "Human-readable name of the light."},
        {"on",   "boolean", "true to turn on, false to turn off."},
    },
};
```

```
from tryll_client.graph import ToolDefinition, ToolParamDefinition

set_light = ToolDefinition(
    name="set_light",
    description="Turn a named light on or off.",
    parameters=[
        ToolParamDefinition(name="name", type="string",
            description="Human-readable name of the light, e.g. 'porch'."),
        ToolParamDefinition(name="on", type="boolean",
            description="true to turn on, false to turn off."),
    ],
)
```

## Step 2 — build the graph with a `ToolCall` node

```
var graph = new TryllGraphBuilder()
    .AddToolCall("detect", new TryllToolCallParams
    {
        Tools                = new List<TryllToolDefinition> { setLight },
        Disposition          = TryllToolCallDisposition.AwaitResult,
        Mode                 = TryllToolCallMode.DetectOnly,
        SystemPrompt         = "You are a smart-home controller.",
        ToolCalledExit       = "answer",
        NoToolCalledExit     = "answer",
    })
    .AddGenerate("answer", new TryllGenerateParams())
    .SetStartNode("detect")
    .SetDefaultModelName("Llama-3.2-3B-Instruct")
    .Build();

var (agent, error) = await TryllClient.Instance.RequestCreateAgentAsync(graph);
```

```
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

ToolCallParamsT detectParams;
detectParams.tools            = {setLight};
detectParams.disposition      = ::Tryll::ToolCallDisposition_AwaitResult;
detectParams.mode             = ::Tryll::ToolCallMode_DetectOnly;
detectParams.system_prompt      = "You are a smart-home controller.";
detectParams.tool_called_exit      = "answer";
detectParams.no_tool_called_exit   = "answer";

GraphDescription graph;
graph.AddToolCall("detect", std::move(detectParams))
     .AddGenerate("answer", GenerateParamsT{})
     .SetStartNode("detect")
     .SetDefaultModelName("Llama-3.2-3B-Instruct");

auto agent = client.CreateAgent(graph);
```

```
from tryll_client.graph import (
    GraphDescription, ToolCallParams, ToolCallMode, GenerateParams,
)
from tryll_client._generated.node_params import ToolCallDisposition

graph = (
    GraphDescription()
    .add_node("detect", ToolCallParams(
        tools=[set_light],
        disposition=ToolCallDisposition.AwaitResult,
        mode=ToolCallMode.DetectOnly,
        system_prompt="You are a smart-home controller.",
        tool_called_exit="answer",
        no_tool_called_exit="answer",
    ))
    .add_node("answer", GenerateParams())
    .set_start_node("detect")
    .set_default_model_name("Llama-3.2-3B-Instruct")
)

agent = client.create_agent(graph)
```

See the full param list in [ToolCall node reference](https://docs.tryllengine.com/reference/nodes/tool-call.html).

## Step 3 — receive the notification client-side

Register a callback before the first `send_message` / `SendMessage` call. Prefer the **call-id-aware** form when you will resume with results — echo that `call_id` in `tool_results`.

```
// Session-level; prefer WithId when returning results.
TryllClient.Instance.ToolCallNotificationWithId +=
    (agentId, callId, toolName, argsJson) => {
        if (toolName == "set_light")
            Debug.Log($"[tool] id={callId} set_light args={argsJson}");
    };

// Or RegisterTool sugar — auto-resumes a complete paused batch.
// RegisterTool handlers only run from the pausing path (HandlePaused),
// i.e. for Pause / PauseAndAcknowledge / AwaitResult. Non-pausing
// notifying dispositions (Notify / NotifyAndAcknowledge) never reach a
// registered handler — use ToolCallNotificationWithId (or the generic
// NodeEvent callback) for those. With disposition=AwaitResult the
// handler return becomes tool_results; Pause / PauseAndAcknowledge run
// the handler for side effects, then Resume() without tool_results
// (sending results would error 3016).
agentComp.RegisterTool("set_light", call => {
    // call.CallId, call.ToolName, call.ArgumentsJson, call.Disposition
    ApplyLight(call.ArgumentsJson);
    return "{\"ok\":true}"; // model-visible only for AwaitResult
});
```

Bind **On Tool Call With Id** on `UTryllSubsystem`, or call `RegisterTool` on the agent / component (auto-resume sugar). Same rule as Unity: `RegisterTool` handlers only run for pausing dispositions (`Pause` / `PauseAndAcknowledge` / `AwaitResult`); native `tool_results` are sent only for `AwaitResult`. Use the typed notification event for `Notify` / `NotifyAndAcknowledge`.

```
AgentComponent->RegisterTool(TEXT("set_light"),
    [](const FTryllToolCall& Call) -> FString
    {
        // Call.CallId, Call.ToolName, Call.ArgumentsJson, Call.Disposition
        return TEXT("{\"ok\":true}");
    });
```

```
agent.SetOnToolCallWithId(
    [&agent](std::string_view callId, std::string_view toolName,
             std::string_view argsJson)
    {
        // Reader thread — keep non-blocking; resume async.
        if (toolName == "set_light")
        {
            AgentProxy::ToolResult tr{std::string{callId}, "{\"ok\":true}"};
            agent.ResumeWithToolResultAsync(tr);
        }
    });
```

Prefer `SetOnToolCallEvent` for new code: it delivers an owning `ToolCallEvent{callId, toolName, argumentsJson, nodeName, disposition}` struct (safe to capture beyond the callback), instead of the `string_view`s above that alias the current frame and must be copied before any deferred use.

```
import json
from tryll_client.agent import ToolResult

def on_tool_call(call_id: str, tool_name: str, arguments_json: str) -> None:
    args = json.loads(arguments_json)
    if tool_name == "set_light":
        # Fires on the reader thread — never call the blocking resume()
        # here (it would deadlock the sole frame reader waiting on its
        # own Ack). resume_async() is safe: it returns a Future and lets
        # the reader thread keep consuming frames.
        agent.resume_async(tool_results=[ToolResult(call_id, '{"ok":true}')])

agent.set_on_tool_call_with_id(on_tool_call)
```

## Step 4 — feed the result back

Protocol v8+ carries results on `ResumeAgentRequest.tool_results`. For `AwaitResult`, resume with a **complete** batch (every pending `call_id` exactly once). Invalid batches return `3016 InvalidToolResults` and leave the turn paused for retry.

```
// Manual resume (if not using RegisterTool):
agentComp.ResumeWithToolResult(
    new TryllToolResult(callId, "{\"ok\":true}"));
```

```
FTryllToolResult Result;
Result.CallId = CallId;
Result.Result = TEXT("{\"ok\":true}");
AgentComponent->ResumeWithToolResult(Result);
```

```
agent.ResumeWithToolResult(
    AgentProxy::ToolResult{std::string{callId}, "{\"ok\":true}"});
```

```
# From a reader-thread tool-call callback, use resume_async (see Step 3).
# From your own thread (e.g. after the callback queued work elsewhere),
# the blocking form is fine:
agent.resume(tool_results=[ToolResult(call_id, '{"ok":true}')])
# tuples also accepted: tool_results=[(call_id, '{"ok":true}')]
```

For side-effect tools that need no payload, keep the default `disposition = Acknowledge` (optional `acknowledge_text`, empty ⇒ `"ok"`) or switch to `RouteOnly`. No `tool_results` required; the projection still closes the exchange so multi-turn templates stay well-formed.

Alternative: ChangeParams for game-only context

If the *model* does not need the tool payload but a downstream node's prompt should mention it for diagnostics, you can still [`ChangeParams`](https://docs.tryllengine.com/how-to/change-agent-parameters.html) while paused and `Resume()` without `tool_results`. Prefer typed `tool_results` whenever the model should condition on the answer.

## Verify it worked

Send a message that should trigger the tool:

```
agentComp.SendMessage("Please turn on the porch light.");
```

Call `UTryllAgentComponent::SendMessage("Please turn on the porch light.")`.

```
agent.SendText("Please turn on the porch light.");
```

```
agent.send_message("Please turn on the porch light.")
```

Your call-id-aware callback fires with something like:

```
call_id        = c00000000
tool_name      = set_light
arguments_json = {"name": "porch", "on": "true"}
```

After a successful `ResumeWithToolResult(s)` / `resume(tool_results=...)`, the downstream `Generate` projects the complete pair and streams the reply.

## Common pitfalls

- **Tool format is automatic.** The legacy `tool_call_format` param is deprecated and ignored. Use a model whose chat template supports tools.
- **Argument values are strings.** Coerce booleans/numbers in your client.
- **Model invents tools.** Allow-list `tool_name` before acting.
- **Non-notifying dispositions** (`RouteOnly`, `Acknowledge`) mean no `tool_call` event — callbacks never fire.
- **`AwaitResult` requires pause + complete results.** Missing/unknown `call_id`s yield `3016` and stay paused.
- **`mode=call_or_answer` (the default)** emits residual text when no tool was detected. Use `detect_only` when a separate `Generate` handles `no_tool_called`. See [Concept: Tool calling](https://docs.tryllengine.com/concepts/tool-calling.html#forcing-a-tool-call).

## Related

- [How to pause and resume a turn](https://docs.tryllengine.com/how-to/pause-and-resume-turns.html)
- [Reference: ToolCall node](https://docs.tryllengine.com/reference/nodes/tool-call.html)
- [Reference: Wire Protocol — Pausing and resuming](https://docs.tryllengine.com/reference/wire-protocol.html#pausing-and-resuming-a-turn)
- [Concept: Tool Calling](https://docs.tryllengine.com/concepts/tool-calling.html)
- [Concept: Workflows, Graphs, and Nodes](https://docs.tryllengine.com/concepts/workflows-graphs-nodes.html)
