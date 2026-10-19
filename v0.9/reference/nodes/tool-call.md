# Tool Call

The Tool Call node uses [language-model](https://docs.tryllengine.com/v0.9/reference/glossary.html#language-model) inference to **detect** whether the model wants to call an external function, given a set of [tool definitions](https://docs.tryllengine.com/v0.9/reference/agent-parameters.html#tooldefinition). It does **not** execute the tool — the server is agnostic to the tool's implementation; the detected call is surfaced to the client.

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

There is no silent pause (event without notify) — use an explicit [`Pause`](https://docs.tryllengine.com/v0.9/reference/nodes/pause.html) node for that.

This detection-only contract makes the node safe to run on-device with no special sandboxing: the server never touches the client's tool implementation.

Like [Generate](https://docs.tryllengine.com/v0.9/reference/nodes/generate.html), the prompt's user turn is selected by `input` (empty = the `user_message` slot; set it to an upstream node's output slot to project that instead — e.g. after a query-rewrite `Generate`).

`NodeType`: `ToolCall`.

Leave `context_size` at `0` to use this node's **2048-token** default (then the model variant, then the server `default_n_ctx`). Authored and BFCL tool-call prompts overflow a 1024 window; 2048 is the product default. Generate still falls back to the variant / server window (commonly 8192). The `context` table is an advanced structural override. Do not set `q4_0` KV expecting a VRAM win at a right-sized window — measured savings show up only at large `n_ctx` (around 8192). `sampling.max_tokens` is fixed at creation and sizes the prompt budget; omitted, it defaults to 758 and must be strictly less than the context size.

## Parameters

| Param                 | Type                 | Default                                         | Range | Structural | Localizable | Description                                                                                                                                                                                                                                                                                                                                                                                       |
| --------------------- | -------------------- | ----------------------------------------------- | ----- | ---------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `model_name`          | str                  | empty                                           | —     | ✓          | —           | Model catalog name. Empty = use the agent's default_model_name.                                                                                                                                                                                                                                                                                                                                   |
| `context_size`        | int                  | 0 -> 2048                                       | ≥ 0.0 | ✓          | —           | KV-cache / context window (n_ctx) for this node in tokens. 0 = node default 2048, else the model variant's context_size, else server default_n_ctx. Validated against the model's trained maximum at agent creation.                                                                                                                                                                              |
| `system_prompt`       | str (multiline)      | empty                                           | —     | —          | ✓           | Prepended before the user turn during projection. Empty = none.                                                                                                                                                                                                                                                                                                                                   |
| `input`               | str                  | empty                                           | —     | ✓          | —           | Slot name this node consumes as its primary text. Empty = "user_message". Structural: immutable after creation — rebinding would re-wire the slot dataflow that is validated once at agent creation.                                                                                                                                                                                              |
| `tool_call_format`    | str                  | empty                                           | —     | ✓          | —           | DEPRECATED (kept for wire/config compatibility; ignored by the server). Tool-call prompting and parsing are now driven by the model's own chat template via llama.cpp common/ (see LlamaCppChatTurn), so no per-dialect format is needed. Structural so mutation is still vetoed.                                                                                                                 |
| `tools`               | list[ToolDefinition] | empty list                                      | —     | ✓          | —           | Callable tool definitions. Structural because the tool-call prompt schema and parser contract are materialised at construction.                                                                                                                                                                                                                                                                   |
| `mode`                | ToolCallMode         | ToolCallMode.CallOrAnswer                       | —     | —          | —           | Tool-choice policy — see Tryll.ToolCallMode. Mutable: the grammar is rebuilt every turn, so a client can flip a node between (for example) RequireCall for a known-command turn and DetectOnly for free chat.                                                                                                                                                                                     |
| `parallel_tool_calls` | bool                 | False                                           | —     | —          | —           | Allow more than one tool call per turn where the model's chat template supports it (most formats cap at one call when false). Mutable for the same reason as `mode`.                                                                                                                                                                                                                              |
| `sampling`            | SamplingOverrides    | unset                                           | —     | —          | —           | Sparse sampling overrides.                                                                                                                                                                                                                                                                                                                                                                        |
| `disposition`         | ToolCallDisposition  | ToolCallDisposition.Acknowledge                 | —     | —          | —           | End-to-end notify / pause / history policy for this node. See Tryll.ToolCallDisposition. Mutable: stamped onto ToolCallRecord at write time, so a mid-session change only affects subsequent turns.                                                                                                                                                                                               |
| `acknowledge_text`    | str                  | empty -> "ok"                                   | —     | —          | ✓           | Result text used when disposition synthesises an acknowledgement (Acknowledge / NotifyAndAcknowledge / PauseAndAcknowledge). Empty = "ok". Ignored for RouteOnly / Notify / Pause / AwaitResult.                                                                                                                                                                                                  |
| `pause_timeout_ms`    | int                  | 0 -> server workflow.tool_call_pause_timeout_ms | ≥ 0.0 | ✓          | —           | Wall-clock budget for a pause this node triggers (pausing dispositions only), in milliseconds. 0 = inherit the server's `workflow.tool_call_pause_timeout_ms` (2000 in the shipped server-config.json; 0 there = infinite). On expiry the turn ends with TurnStatus.PauseTimedOut; there is no timeout exit on this node. Structural: resolved once against the server default at agent creation. |
| `context`             | ContextOverrides     | unset                                           | —     | ✓          | —           | Backend context construction. Null table inherits, except CreateContext still defaults `n_outputs_max` to 1. Structural. Appended after pause_timeout_ms so existing field IDs do not move.                                                                                                                                                                                                       |
| `adaptation`          | ModelAdaptation      | unset                                           | —     | ✓          | —           | Changes how the model behaves for this node (LoRA adapters; later control vectors). Null = base model. Structural: applied to the llama context at creation, so a change means a new context. Appended last so existing field IDs do not move.                                                                                                                                                    |

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

An empty exit field routes to END and is valid. A non-empty exit that does not name a node in the graph fails agent creation with [`3003 GraphCompilationFailed`](https://docs.tryllengine.com/v0.9/reference/error-codes.html#3xxx-agent).

## Side effects

- Records **every** detected tool call on the current [turn](https://docs.tryllengine.com/v0.9/reference/glossary.html#interaction) (one record per call; a single turn may produce more than one).
- When `mode = call_or_answer` and no tool is detected, additionally appends the model's text as the assistant's reply and fires `AnswerText` chunks (as a [Generate node](https://docs.tryllengine.com/v0.9/reference/nodes/generate.html) would).
- When `mode = require_call`, the model is forced to call a tool every turn (no plain-text reply is possible) — see [Concept: Tool calling](https://docs.tryllengine.com/v0.9/concepts/tool-calling.html#forcing-a-tool-call) for when this is (and isn't) appropriate.
- When `disposition` notifies (`Notify`, `NotifyAndAcknowledge`, `Pause`, `PauseAndAcknowledge`, `AwaitResult`), fires one `NodeEvent` (`event_type="tool_call"`) per detected call (`tool_name`, `arguments_json`, `call_id`, `disposition`), out-of-band, before the turn completes. `call_id` is a nine-character alphanumeric correlation id (`c%08x`) for `tool_results`.
- When `disposition` synthesises history (`Acknowledge`, `NotifyAndAcknowledge`, `PauseAndAcknowledge`), writes a synthetic `ToolResultComponent` immediately so later projection is already a complete pair (text from `acknowledge_text`, empty ⇒ `"ok"`).
- When `disposition` pauses (`Pause`, `PauseAndAcknowledge`, `AwaitResult`) and the node exits `tool_called`, the executor pauses so the client can `ChangeAgentParam` and/or return `tool_results` before `ResumeAgentRequest` continues. No pause on `no_tool_called`. See [How to pause and resume a turn](https://docs.tryllengine.com/v0.9/how-to/pause-and-resume-turns.html).
- A pause this node triggers can time out. `pause_timeout_ms = 0` inherits the server's `workflow.tool_call_pause_timeout_ms`, which is **2000 ms** in the shipped `server-config.json`; the pause waits forever only when that server value is `0`. On expiry the turn ends with `TurnStatus.PauseTimedOut` — there is no timeout exit on this node.
- Diagnostics include `disposition` and (when set) `acknowledge_text`.

## Tool-call formats

Tool prompting and parsing are handled automatically by the model's own chat template (via llama.cpp `common/`): tool definitions are rendered the way the model was trained, generation is constrained by a grammar so calls are well-formed, and the output is parsed back into structured calls. You no longer choose a dialect — the legacy `tool_call_format` param is **deprecated and ignored**.

## Diagnostics

When `enable_diagnostics = true`, this node contributes to `debug_info.nodes[].diagnostics`. See [Turn Diagnostics JSON](https://docs.tryllengine.com/v0.9/reference/turn-diagnostics-json.html) for the envelope, the shared `input.prompt[]` shape, and the `engine` counters.

| Key                                                                                                                                  | Meaning                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Absent when                                        |
| ------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| `parameters.model_name`                                                                                                              | The model that actually ran, after fallback.                                                                                                                                                                                                                                                                                                                                                                                                            | The name could not be resolved.                    |
| `parameters.tool_count`                                                                                                              | Number of tool definitions passed to the model.                                                                                                                                                                                                                                                                                                                                                                                                         | —                                                  |
| `parameters.tools[]`                                                                                                                 | Flattened echo of what the model was offered — one signature line per tool (`search(query: string) - Search the web`), then an indented `query - What to search for` line per described parameter. Carries your authored descriptions, so content-free telemetry sinks strip it.                                                                                                                                                                        | —                                                  |
| `parameters.mode`                                                                                                                    | `detect_only`, `call_or_answer` or `require_call`.                                                                                                                                                                                                                                                                                                                                                                                                      | —                                                  |
| `parameters.parallel_tool_calls`                                                                                                     | Whether the model was allowed to emit several calls at once.                                                                                                                                                                                                                                                                                                                                                                                            | —                                                  |
| `parameters.disposition`                                                                                                             | `route_only`, `acknowledge`, `notify`, `await_result`, …                                                                                                                                                                                                                                                                                                                                                                                                | —                                                  |
| `parameters.acknowledge_text`                                                                                                        | The acknowledgement string.                                                                                                                                                                                                                                                                                                                                                                                                                             | Not set.                                           |
| `parameters.input`                                                                                                                   | The configured input slot name.                                                                                                                                                                                                                                                                                                                                                                                                                         | Using the `user_message` default.                  |
| `parameters.temperature`, `top_p`, `top_k`, `min_p`, `max_tokens`, `seed`, `repeat_penalty`, `presence_penalty`, `frequency_penalty` | The resolved sampling set.                                                                                                                                                                                                                                                                                                                                                                                                                              | —                                                  |
| `input.prompt[]`                                                                                                                     | The rendered prompt, including any historical assistant `tool_calls`.                                                                                                                                                                                                                                                                                                                                                                                   | —                                                  |
| `output.text`                                                                                                                        | Plain content the model produced alongside (or instead of) a call.                                                                                                                                                                                                                                                                                                                                                                                      | The model produced none.                           |
| `output.reasoning_content`                                                                                                           | Extracted chain-of-thought.                                                                                                                                                                                                                                                                                                                                                                                                                             | The model emitted none.                            |
| `output.tool_calls[]`                                                                                                                | Each detected call: `name`, `arguments_json` (raw, as the model wrote it), and `id` when the model supplied one.                                                                                                                                                                                                                                                                                                                                        | No call was detected.                              |
| `output.parse_fallback`                                                                                                              | `true` when the output could not be parsed through the model's chat template. No call is recorded, `output.text` holds the raw output, and the node exits `no_tool_called`. Under `mode = call_or_answer` that raw text is sent as the answer. Can also be `true` when the model answered in prose instead of calling — `output.parse_failure` tells the two apart.                                                                                     | —                                                  |
| `output.parse_failure`                                                                                                               | Why the parse failed. `malformed_call`: the output contains the marker that starts a call in the model's own syntax (`[TOOL_CALLS]` on Mistral-family models), but the call does not parse. `no_call`: no marker, so the model answered in prose. `unclassified`: the chat template declares no marker to check against.                                                                                                                                | `parse_fallback` is `false`.                       |
| `output.parse_error`                                                                                                                 | The parser's reason and where it gave up, as a byte offset into `output.text` with up to 48 bytes of the text from there — for example `…does not match the expected peg-native format; unparsed from byte 75: "[TOOL_CALLS] [{"name": …"` for a second call while `parallel_tool_calls` is `false`. The offset is often the start of the call that failed, not the exact bad character. Quotes model output, so content-free telemetry sinks strip it. | `parse_fallback` is `false`.                       |
| `engine.*`                                                                                                                           | Scheduler counters plus `chat_turn` — see below.                                                                                                                                                                                                                                                                                                                                                                                                        | Server config `include_engine_diagnostics` is off. |

### Diagnosing a call that never happened

Three places answer "why didn't the model call anything", in order:

1. **`engine.chat_turn.has_grammar`** — `false` with `mode: require_call` means the sampler was never constrained, and the turn ran free to `max_tokens`. `grammar_lazy` says the grammar only arms once the model starts a call.
1. **`output.parse_fallback`** — `true` means the model's output could not be parsed as a call, so the call was dropped. `output.parse_failure: malformed_call` means the model tried and got the syntax wrong; `output.parse_error` says where the parser stopped. `no_call` means the model answered in prose.
1. **`parameters.tools[]`** — confirms the descriptions the model actually saw. See [Design a tool-call prompt](https://docs.tryllengine.com/v0.9/how-to/design-a-tool-call-prompt.html).

If the player sees tool-call markup such as `<|tool_call>…` in the answer, the text may come from this node or from a speech node after it. See [Fix tool-call text in answers](https://docs.tryllengine.com/v0.9/how-to/fix-tool-call-text-in-answers.html).

### Where the results are

The node's `output.tool_calls[]` is the model's side of the exchange. The environment's answer is not here:

- The turn-level [`tool_calls[]`](https://docs.tryllengine.com/v0.9/reference/turn-diagnostics-json.html#tool-calls) gives a flat `name` + `params` summary of every call the turn emitted.
- The result text, the dispatch state and the exchange pairing live in the turn's [`interaction`](https://docs.tryllengine.com/v0.9/reference/turn-diagnostics-json.html#interaction) snapshot, as `ToolCallRecord` and `ToolResultComponent` entries linked by `call_id`. That section is present only when the server enables `include_interaction_in_diagnostics`, which is why the turn inspector's tool-call block can show calls with no results.

A `disposition` that does not return results to the model leaves the result side genuinely empty — that is configuration, not a missing diagnostic.

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

Or author `UTryllToolCallParams` (with tool definitions) inside a `UTryllWorkflowAsset`; bind `UTryllAgentComponent::OnToolCall` to receive the detection. `UTryllSubsystem::OnToolCall` is the all-agents form.

Receive the notification in your client with `agent.set_on_tool_call(cb)` (Python), `agent.SetOnToolCall(cb)` (C++), `TryllAgentComponent.ToolCallNotification` (Unity), or `UTryllAgentComponent::OnToolCall` (Unreal). Session events stay available when one listener should see every agent.

See the full flow (including client-side execution and feeding the result back) in [How to define and handle tool calls](https://docs.tryllengine.com/v0.9/how-to/define-and-handle-tool-calls.html).

## Client bindings

- **C++:** `GraphDescription::AddToolCall(name, ToolCallParamsT)` — `GraphDescription.h`
- **Python:** `GraphDescription.add_node(name, ToolCallParams(...))` — `tryll_client.graph`
- **Unity:** `TryllGraphBuilder.AddToolCall(name, new TryllToolCallParams{...})` — `Runtime/Generated/TryllGraphBuilder.Nodes.cs`
- **Unreal:** `AddToolCallNode(builder, name, UTryllToolCallParams*)` — `Generated/TryllGraphBuilder.Nodes.h`

## Related

- [Concept: Tool calling](https://docs.tryllengine.com/v0.9/concepts/tool-calling.html)
- [How to define and handle tool calls](https://docs.tryllengine.com/v0.9/how-to/define-and-handle-tool-calls.html)
- [How to test tool calls in Chat and the Dialog Lab](https://docs.tryllengine.com/v0.9/how-to/test-tool-calls-in-editor.html)
- [How to pause and resume a turn](https://docs.tryllengine.com/v0.9/how-to/pause-and-resume-turns.html)
- [Reference: Pause node](https://docs.tryllengine.com/v0.9/reference/nodes/pause.html)
- [Agent Parameters — `ToolDefinition`](https://docs.tryllengine.com/v0.9/reference/agent-parameters.html#tooldefinition)
- [Glossary](https://docs.tryllengine.com/v0.9/reference/glossary.html#tool-call)
