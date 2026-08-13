# Tool Call

The Tool Call node uses [language-model](https://docs.tryllengine.com/reference/glossary.html#language-model) inference to **detect** whether the model wants to call an external function, given a set of [tool definitions](https://docs.tryllengine.com/reference/agent-parameters.html#tooldefinition). It does **not** execute the tool — the server is agnostic to the tool's implementation; the detected call is surfaced to the client.

How (or whether) a result re-enters the model is controlled by `disposition` (`ToolCallDisposition`):

| Disposition                 | Event | Pause | History               |
| --------------------------- | ----- | ----- | --------------------- |
| `RouteOnly`                 | No    | No    | Omit                  |
| `Acknowledge` (**default**) | No    | No    | Synthetic ack         |
| `Notify`                    | Yes   | No    | Omit                  |
| `NotifyAndAcknowledge`      | Yes   | No    | Synthetic ack         |
| `Pause`                     | Yes   | Yes   | Omit                  |
| `PauseAndAcknowledge`       | Yes   | Yes   | Synthetic ack         |
| `AwaitResult`               | Yes   | Yes   | Client `tool_results` |

There is no silent pause (event without notify) — use an explicit [`Pause`](https://docs.tryllengine.com/reference/nodes/pause.html) node for that.

This detection-only contract makes the node safe to run on-device with no special sandboxing: the server never touches the client's tool implementation.

Like [Generate](https://docs.tryllengine.com/reference/nodes/generate.html), the prompt's user turn is selected by `input` (empty = the `user_message` slot; set it to an upstream node's output slot to project that instead — e.g. after a query-rewrite `Generate`).

`NodeType`: `ToolCall`.

## Parameters

| Param                 | Type                      | Default                         | Range | Structural | Description                                                                                                                                                                                                                                                                       |
| --------------------- | ------------------------- | ------------------------------- | ----- | ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `model_name`          | Optional[str]             | inherit model default           | —     | ✓          | Model catalog name. Empty = use the agent's default_model_name.                                                                                                                                                                                                                   |
| `context_size`        | int                       | 0                               | ≥ 0.0 | ✓          | KV-cache / context window (n_ctx) for this node in tokens. 0 = fall back to the model variant's context_size, else the server default_n_ctx. Validated against the model's trained maximum at agent creation.                                                                     |
| `system_prompt`       | Optional[str] (multiline) | inherit model default           | —     | —          | Prepended before the user turn during projection.                                                                                                                                                                                                                                 |
| `input`               | Optional[str]             | inherit model default           | —     | ✓          | Slot name this node consumes as its primary text. Empty = "user_message". Structural: immutable after creation — rebinding would re-wire the slot dataflow that is validated once at agent creation.                                                                              |
| `tool_call_format`    | Optional[str]             | inherit model default           | —     | ✓          | DEPRECATED (kept for wire/config compatibility; ignored by the server). Tool-call prompting and parsing are now driven by the model's own chat template via llama.cpp common/ (see LlamaCppChatTurn), so no per-dialect format is needed. Structural so mutation is still vetoed. |
| `tools`               | Optional[Any]             | inherit model default           | —     | ✓          | Callable tool definitions. Structural because the tool-call prompt schema and parser contract are materialised at construction.                                                                                                                                                   |
| `mode`                | ToolCallMode              | ToolCallMode.CallOrAnswer       | —     | —          | Tool-choice policy — see Tryll.ToolCallMode. Mutable: the grammar is rebuilt every turn, so a client can flip a node between (for example) RequireCall for a known-command turn and DetectOnly for free chat.                                                                     |
| `parallel_tool_calls` | bool                      | False                           | —     | —          | Allow more than one tool call per turn where the model's chat template supports it (most formats cap at one call when false). Mutable for the same reason as `mode`.                                                                                                              |
| `sampling`            | Optional[Any]             | inherit model default           | —     | —          | Sparse sampling overrides.                                                                                                                                                                                                                                                        |
| `disposition`         | ToolCallDisposition       | ToolCallDisposition.Acknowledge | —     | —          | End-to-end notify / pause / history policy for this node. See Tryll.ToolCallDisposition. Mutable: stamped onto ToolCallRecord at write time, so a mid-session change only affects subsequent turns.                                                                               |
| `acknowledge_text`    | Optional[str]             | empty -> "ok"                   | —     | —          | Result text used when disposition synthesises an acknowledgement (Acknowledge / NotifyAndAcknowledge / PauseAndAcknowledge). Empty = "ok". Ignored for RouteOnly / Notify / Pause / AwaitResult.                                                                                  |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit             | Param field           | Description                                                                                                                                                           |
| ---------------- | --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tool_called`    | `tool_called_exit`    | Exit taken when one or more tool calls are parsed from the response. Empty string = END.                                                                              |
| `no_tool_called` | `no_tool_called_exit` | Exit taken when no tool call is parsed (or, under mode=call_or_answer, when the model's residual text was emitted as a plain-text reply instead). Empty string = END. |

## Exit routes

| Route            | Fires when                                                 |
| ---------------- | ---------------------------------------------------------- |
| `tool_called`    | At least one tool call was detected in the model's output. |
| `no_tool_called` | The model produced no parseable tool call.                 |

Both routes must be wired.

## Side effects

- Records **every** detected tool call on the current [turn](https://docs.tryllengine.com/reference/glossary.html#interaction) (one record per call; a single turn may produce more than one).
- When `mode = call_or_answer` and no tool is detected, additionally appends the model's text as the assistant's reply and fires `AnswerText` chunks (as a [Generate node](https://docs.tryllengine.com/reference/nodes/generate.html) would).
- When `mode = require_call`, the model is forced to call a tool every turn (no plain-text reply is possible) — see [Concept: Tool calling](https://docs.tryllengine.com/concepts/tool-calling.html#forcing-a-tool-call) for when this is (and isn't) appropriate.
- When `disposition` notifies (`Notify`, `NotifyAndAcknowledge`, `Pause`, `PauseAndAcknowledge`, `AwaitResult`), fires one `NodeEvent` (`event_type="tool_call"`) per detected call (`tool_name`, `arguments_json`, `call_id`, `disposition`), out-of-band, before the turn completes. `call_id` is a nine-character alphanumeric correlation id (`c%08x`) for `tool_results`.
- When `disposition` synthesises history (`Acknowledge`, `NotifyAndAcknowledge`, `PauseAndAcknowledge`), writes a synthetic `ToolResultComponent` immediately so later projection is already a complete pair (text from `acknowledge_text`, empty ⇒ `"ok"`).
- When `disposition` pauses (`Pause`, `PauseAndAcknowledge`, `AwaitResult`) and the node exits `tool_called`, the executor pauses so the client can `ChangeAgentParam` and/or return `tool_results` before `ResumeAgentRequest` continues. No pause on `no_tool_called`. See [How to pause and resume a turn](https://docs.tryllengine.com/how-to/pause-and-resume-turns.html).
- Diagnostics include `disposition` and (when set) `acknowledge_text`.

## Tool-call formats

Tool prompting and parsing are handled automatically by the model's own chat template (via llama.cpp `common/`): tool definitions are rendered the way the model was trained, generation is constrained by a grammar so calls are well-formed, and the output is parsed back into structured calls. You no longer choose a dialect — the legacy `tool_call_format` param is **deprecated and ignored**.

## Diagnostics

When `enable_diagnostics = true`, the node contributes these keys to `TurnComplete.debug_info`:

| Key                                                                    | Meaning                                                                                                                                                                                                                                                                   |
| ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tool_count`                                                           | Number of tool definitions passed to the model.                                                                                                                                                                                                                           |
| `tools`                                                                | Flattened echo of the tools the model was offered — one signature line per tool (`search(query: string) - Search the web`), followed by an indented `query - What to search for` line for each parameter that has a description. Omitted by content-free telemetry sinks. |
| `mode`                                                                 | Current value of the param (`detect_only` / `call_or_answer` / `require_call`).                                                                                                                                                                                           |
| `parallel_tool_calls`                                                  | Current value of the param.                                                                                                                                                                                                                                               |
| `disposition`                                                          | Current value (`route_only` / `acknowledge` / `notify` / …).                                                                                                                                                                                                              |
| `acknowledge_text`                                                     | Present when non-empty.                                                                                                                                                                                                                                                   |
| `input`                                                                | The configured `input` slot name, when set (absent when using the `user_message` default).                                                                                                                                                                                |
| `reasoning_content`                                                    | Extracted chain-of-thought, when the model emitted any.                                                                                                                                                                                                                   |
| (plus sampling / prompt diagnostics shared with other inference nodes) | —                                                                                                                                                                                                                                                                         |

## Minimum working example

```
from tryll_client.graph import (
    GraphDescription, ToolCallParams, GenerateParams,
    ToolDefinition, ToolParamDefinition,
)
from tryll_client._generated.node_params import ToolCallDisposition

tools = [
    ToolDefinition(
        name="get_weather",
        description="Get the current weather for a city.",
        parameters=[
            ToolParamDefinition(name="city", type="string",
                description="City name"),
        ],
    ),
]

graph = (
    GraphDescription()
    .add_node("detect", ToolCallParams(
        tools=tools,
        disposition=ToolCallDisposition.NotifyAndAcknowledge,
        tool_called_exit="",    # empty = END (client handles the tool)
        no_tool_called_exit="answer",
    ))
    .add_node("answer", GenerateParams(
        default_exit="",   # empty = END
    ))
    .set_start_node("detect")
    .set_default_model_name("Qwen2.5-3B-Instruct")
)

agent = client.create_agent(graph)
```

```
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

auto cityParam = std::make_unique<ToolParamDefinitionT>();
cityParam->name        = "city";
cityParam->type        = "string";
cityParam->description = "City name";

auto getWeather = std::make_unique<ToolDefinitionT>();
getWeather->name        = "get_weather";
getWeather->description = "Get the current weather for a city.";
getWeather->parameters.push_back(std::move(cityParam));

ToolCallParamsT dp;
dp.disposition       = ::Tryll::ToolCallDisposition_NotifyAndAcknowledge;
dp.tool_called_exit  = "";       // empty = END (client handles the tool)
dp.no_tool_called_exit = "answer";
dp.tools.push_back(std::move(getWeather));

GenerateParamsT answerP;
// answerP.default_exit = ""; // empty = END (the default)

GraphDescription graph;
graph.AddToolCall("detect", std::move(dp))
     .AddGenerate("answer", std::move(answerP))
     .SetStartNode("detect")
     .SetDefaultModelName("Qwen2.5-3B-Instruct");

auto agent = client.CreateAgent(graph);
```

```
using Tryll.Client;
using System.Collections.Generic;

var tools = new List<TryllToolDefinition>
{
    new TryllToolDefinition
    {
        Name        = "get_weather",
        Description = "Get the current weather for a city.",
        Parameters  = new List<TryllToolParamDefinition>
        {
            new TryllToolParamDefinition
                { Name = "city", Type = "string", Description = "City name" },
        },
    },
};

var graph = new TryllGraphBuilder()
    .AddToolCall("detect", new TryllToolCallParams
    {
        Tools           = tools,
        Disposition     = TryllToolCallDisposition.NotifyAndAcknowledge,
        ToolCalledExit  = "",       // empty = END (client handles the tool)
        NoToolCalledExit = "answer",
    })
    .AddGenerate("answer", new TryllGenerateParams())
    .SetStartNode("detect")
    .SetDefaultModelName("Qwen2.5-3B-Instruct")
    .Build();
```

```
#include "Generated/TryllGraphBuilder.Nodes.h"
#include "Generated/TryllNodeParamsFactory.h"

UTryllToolCallParams* DetectP = UTryllNodeParamsFactory::MakeToolCallParams(this);
DetectP->Disposition     = ETryllToolCallDisposition::NotifyAndAcknowledge;
DetectP->ToolCalledExit  = TEXT("");       // empty = END
DetectP->NoToolCalledExit = TEXT("answer");
// Author tool definitions via DetectP->Tools (TArray<FTryllToolDefinition>)
FTryllToolDefinition GetWeather;
GetWeather.Name        = TEXT("get_weather");
GetWeather.Description = TEXT("Get the current weather for a city.");
GetWeather.Parameters.Add({ TEXT("city"), TEXT("string"), TEXT("City name") });
DetectP->Tools.Add(GetWeather);

FTryllGraphDescription Graph = FTryllGraphBuilder()
    .AddNode(TEXT("detect"), DetectP)
    .AddNode(TEXT("answer"), UTryllNodeParamsFactory::MakeGenerateParams(this))
    .SetStartNode(TEXT("detect"))
    .SetDefaultModelName(TEXT("Qwen2.5-3B-Instruct"))
    .Build();
```

Or author `UTryllToolCallParams` (with tool definitions) inside a `UTryllWorkflowAsset`; bind `UTryllSubsystem::OnToolCall` to receive the detection.

Receive the notification in your client with `agent.set_on_tool_call(cb)` (Python), `agent.SetOnToolCall(cb)` (C++), or `UTryllSubsystem::OnToolCall` (Unreal).

See the full flow (including client-side execution and feeding the result back) in [How to define and handle tool calls](https://docs.tryllengine.com/how-to/define-and-handle-tool-calls.html).

## Client bindings

- **C++:** `GraphDescription::AddToolCall(name, ToolCallParamsT)` — `GraphDescription.h`
- **Python:** `GraphDescription.add_node(name, ToolCallParams(...))` — `tryll_client.graph`
- **Unity:** `TryllGraphBuilder.AddToolCall(name, new TryllToolCallParams{...})` — `Runtime/Generated/TryllGraphBuilder.Nodes.cs`
- **Unreal:** `AddToolCallNode(builder, name, UTryllToolCallParams*)` — `Generated/TryllGraphBuilder.Nodes.h`

## Related

- [Concept: Tool calling](https://docs.tryllengine.com/concepts/tool-calling.html)
- [How to define and handle tool calls](https://docs.tryllengine.com/how-to/define-and-handle-tool-calls.html)
- [How to pause and resume a turn](https://docs.tryllengine.com/how-to/pause-and-resume-turns.html)
- [Reference: Pause node](https://docs.tryllengine.com/reference/nodes/pause.html)
- [Agent Parameters — `ToolDefinition`](https://docs.tryllengine.com/reference/agent-parameters.html#tooldefinition)
- [Glossary](https://docs.tryllengine.com/reference/glossary.html#tool-call)
