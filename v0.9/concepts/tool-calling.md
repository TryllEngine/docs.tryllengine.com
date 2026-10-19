# Tool Calling

"Tool calling" is the trick that turns a chat model into something that can act on the world — look up the weather, move a camera, trigger a quest. In Tryll, tool calling is **detection-based**: the server tells your client *that* a tool should be called and *with what arguments*, and your client decides whether to actually run it — the server never executes the tool itself. This page explains why that split exists, how small local models are coaxed into producing tool calls, and where the edges are.

## The shape of a tool call in Tryll

```
sequenceDiagram
    participant C as Client
    participant S as Server
    participant TC as ToolCall node
    participant M as Model

    C->>S: CreateAgent(graph with ToolCall node + tools[])
    C->>S: SendMessage("turn on the porch light")
    S->>TC: Execute
    TC->>M: Generate (non-streaming) with tool-schema prompt
    M-->>TC: "{"tool": "set_light", "args": {"name":"porch","on":true}}"
    TC-->>S: parsed call + ToolCallNotification (if disposition notifies)
    S-->>C: ToolCallNotification{tool_name, arguments_json}
    TC->>TC: record the call on the current turn
    TC-->>S: exit "tool_called"
    Note over S,C: pause + Resume(tool_results) when AwaitResult;<br/>else Acknowledge / RouteOnly close history without a result
```

Three things to notice:

1. The **model never talks to tools directly.** It only produces text. The node parses that text for a tool-call pattern.
1. The server **never executes the tool.** It fires a notification to the client and moves on. For model-visible results use `disposition = AwaitResult` and resume with `tool_results`; for side effects use `Acknowledge` / `RouteOnly` / other non-`AwaitResult` dispositions.
1. A **failed parse is not an error.** If no tool pattern is found, the node takes the `no_tool_called` route. That is a perfectly normal control flow — branch the graph on it.

## Why detection-only?

Small local models in 2025 are not uniformly good at tool calls. Different model families were fine-tuned on different prompt shapes and different output grammars. Crucially, none of them know about your tools.

Running the tool server-side would force Tryll to:

- sandbox arbitrary code,
- make network / filesystem policy decisions for your app, and
- embed a universal tool-dispatcher that can never match the specifics of a game engine or a desktop app.

Detection-only punts all of that back to the client, where the code to "turn on the porch light" already lives. The server's job is to reliably extract `{tool_name, arguments}` from the model — which is the hard part in small models anyway.

It also means tool calls *compose* naturally with the rest of Tryll. A [graph](https://docs.tryllengine.com/v0.9/concepts/workflows-graphs-nodes.html) can put a [guardrail](https://docs.tryllengine.com/v0.9/reference/nodes/regex-guardrail.html) in front of a `ToolCall` node, a `CannedResponse` on the rejection path, and so on. Nothing about the graph structure is special because tools are in it.

## Prompting is driven by the model's own chat template

You do **not** pick a tool-call "format" in Tryll. Tool prompting and parsing flow through each model's own chat template (via llama.cpp's `common/` engine): the model's GGUF decides where the tool schema goes in the prompt, how each tool definition is rendered, and what pattern its tool calls take. Tryll advertises the tools, the template renders them, the returned grammar constrains generation, and the same template's parser extracts `{tool_name, arguments}` from the output — the exact syntax the model was trained on (Qwen's `<tool_call>` XML, Granite's `<|tool_call|>` block, Llama-3's JSON, …).

This replaced an earlier hand-rolled system of four fixed formats (`chatml` / `llama3` / `mistral` / `generic`) selected via a `tool_call_format` param. That param still exists on the `ToolCall` node for wire/config compatibility but is **deprecated and ignored** by the server — do not set it. If a converted GGUF's embedded template doesn't render tools even though the model was trained for them, a server admin can override it per-variant with `chat_template_file` (see [Model Management](https://docs.tryllengine.com/v0.9/reference/model-management.html#variants-entry-fields)), and `CreateAgent` fails fast when a template can't render tools at all (see [Not every model can call tools](#not-every-model-can-call-tools)).

## What the node actually does

On each turn the `ToolCall` node:

1. Advertises the tool schema to the model through its **own chat template** (which decides where the schema goes and how each tool is rendered).
1. Runs a **non-streaming** `Generate` against the node's model, with the template-provided grammar constraining generation. Temperature is typically low for tool calls (the default is to inherit sampling params; override them to 0 for determinism).
1. Parses the output with the same template's parser to extract content, reasoning, and any tool calls — in whatever syntax that model was trained on.
1. For each parsed call, records the call on the current turn. If `disposition` notifies, also sends a `ToolCallNotification` wire frame so your client can react.
1. Exits via `tool_called` (one or more parsed) or `no_tool_called`.

One param worth calling out: **`mode`**. It replaces the two independent ideas of "what if no tool call comes back" and "should a call even be optional" with a single three-way choice:

| `mode`                     | When no tool is called                                              | Use when                                                                                                      |
| -------------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `call_or_answer` (default) | The model's residual text is emitted as a normal `AnswerText` reply | Your graph puts the `ToolCall` node *in place of* a `Generate` node — one node either calls a tool or answers |
| `detect_only`              | Nothing is emitted; the turn just routes `no_tool_called`           | A separate `Generate` node takes over after `no_tool_called`                                                  |
| `require_call`             | Not applicable — a call is mandatory                                | See [Forcing a tool call](#forcing-a-tool-call) below                                                         |

`mode` is mutable — see [How to change agent parameters](https://docs.tryllengine.com/v0.9/how-to/change-agent-parameters.html) — so a node can flip between `require_call` (for a turn you know is a command) and `detect_only`/`call_or_answer` (for free chat) without recreating the agent.

### Forcing a tool call

Some models — notably ones fine-tuned on the Mistral tool-call format, including Mistral 7B (with the `chat_template_file` override) and the Ministral 3 models — often "know" the right tool and arguments but simply never emit the format's trigger token under normal generation. The underlying reason is that the tool-call grammar is *lazy* by default: it only starts constraining the model's output once the model has already committed to calling a tool on its own. If it never does, generation drifts into plain text and the turn silently routes `no_tool_called`.

`mode = require_call` switches the grammar to *eager*: it constrains output from the very first token and guarantees at least one call. This reliably fixes the "knows the answer, never emits the tag" failure — but it comes with real trade-offs, so use it deliberately:

- **The model cannot answer plain questions anymore.** If a turn isn't actually a command, `require_call` still forces a tool call — usually with `"unspecified"`-style placeholder arguments. Only use `require_call` behind something that already knows the turn is a command (an intent/routing node, or client-side knowledge of the input channel), and switch back to `detect_only` / `call_or_answer` for anything else.
- **It can occasionally stall instead of failing fast.** On some models, a turn the model is reluctant to act on can run all the way to `max_tokens` before giving up, rather than immediately emitting a placeholder call. Budget `max_tokens` accordingly for `require_call` nodes.
- **It does not fix a template that can't render tools at all** — that is the [fail-fast check below](#not-every-model-can-call-tools), a separate problem `require_call` cannot work around.
- **Multi-turn Mistral-family tool calling is supported** under the default complete-pair history projection (`Acknowledge` / `RouteOnly` / `AwaitResult` — see below). An earlier dangling-call shape used to crash those templates on the second tool-calling turn; that gap is closed.

Node creation rejects `mode = require_call` combined with an empty `tools` list up front (forcing a call with nothing to call is meaningless).

## Tool-call history: complete pairs, never dangling

Once a `ToolCall` node detects a call, the turn typically has no `AnswerText` — only tool-call records (and, depending on policy, tool results). Tryll projects those into later prompts as a **complete** `assistant(tool_calls)` → `tool(result)` pair in the model's native syntax — never a half-pair dangling before the next user turn (that shape was off-distribution and crashed Mistral-family templates).

`ToolCallParams.disposition` chooses notify, pause, and history shape (schema default **`Acknowledge`**):

| Disposition                     | History                                                      | Client result?                             |
| ------------------------------- | ------------------------------------------------------------ | ------------------------------------------ |
| `RouteOnly`                     | Omit tool syntax; empty assistant closer on historical turns | No                                         |
| `Acknowledge`                   | Complete pair with `acknowledge_text` (empty ⇒ `"ok"`)       | No                                         |
| `Notify`                        | Omit                                                         | No (event only)                            |
| `NotifyAndAcknowledge`          | Synthetic ack                                                | No (event only)                            |
| `Pause` / `PauseAndAcknowledge` | Omit / synthetic ack                                         | No — pause for `ChangeParams`, not results |
| `AwaitResult`                   | Complete pair with your `ResumeAgentRequest.tool_results`    | Yes — pause required                       |

Use `AwaitResult` when the model must condition on a real tool payload: pause after the call, run the tool client-side, resume with results keyed by `call_id`, then let a downstream `Generate` speak. Unity and Unreal deliver the call on the agent component (`ToolCallNotification` / `OnToolCall`, plus the call-id and detailed forms). The session events on `TryllClient` and `UTryllSubsystem` are the all-agents form and still lead with the agent id. `RegisterTool` can auto-resume, but its handlers **only run for pausing dispositions** (`Pause`, `PauseAndAcknowledge`, `AwaitResult`) — they never fire for `Notify` / `NotifyAndAcknowledge`, which have no pause to auto-resume from. Among the dispositions a handler does run for, the client only attaches `tool_results` when the batch is `AwaitResult` (see `disposition` on the `tool_call` event); `Pause` / `PauseAndAcknowledge` auto-resume without a result payload. For `Notify` / `NotifyAndAcknowledge`, react via the typed call-ID notification event (or the generic `NodeEvent` callback) instead of `RegisterTool`. C++/Python use `ResumeWithToolResult(s)` / `resume(..., tool_results=)` (prefer the `*Async` / `resume_async` forms from a reader-thread callback). See [Define and handle tool calls](https://docs.tryllengine.com/v0.9/how-to/define-and-handle-tool-calls.html).

Prose summaries ("I called operate_door…") are deliberately **not** offered — strong instruction-followers imitate that text instead of emitting a real call next turn.

## Not every model can call tools

Advertising tools only works if the model's chat template actually renders them — and several shipped GGUF conversions simply don't (their template ignores the `tools` variable entirely, so the model never even sees that a tool exists). Running a `ToolCall` node against one of those models used to fail silently: every turn routed `no_tool_called` with no diagnostic.

`CreateAgent` now rejects a `ToolCall` node up front when its model can't render tool definitions, with an error naming the model and suggesting a fix:

```
tool calling is not supported for model 'Gemma 3 4B Instruct (Q4_K_M)'
because its chat template does not render tool definitions. Set
'chat_template_file' on the model's variant in models.json to an
override template with tool support, or remove the tools from this node.
```

If the model genuinely supports tool calling but the *converted* GGUF's embedded template doesn't render it (a real gap — the model itself may still have been trained on a tool format), a server admin can point `chat_template_file` at a replacement Jinja template for that variant. See [Model Management: `variants` entry fields](https://docs.tryllengine.com/v0.9/reference/model-management.html#variants-entry-fields).

To see the verdict *before* downloading a model, check the selected model's detail pane in the Unity/Unreal Model Manager: every language model in the catalog declares an advisory `tool_call_support` value (`supported` or `unsupported`) per variant. It is a hint for picking a model, not a guarantee — the fail-fast check above is what actually decides at agent creation.

## What makes tool calls reliable (or unreliable)

From most to least impactful:

1. **Pick a model whose chat template supports tools.** This is the biggest lever — a model trained and templated for tool calls (Qwen 3.x, Granite 4.x, Llama-3.x, …) is far more reliable than one coaxed into it. `CreateAgent` rejects a model whose template can't render tools at all.
1. **Keep the tool list short.** Small models get overwhelmed past 5–8 tools. If you have many, put a routing / guardrail node in front to select a subset.
1. **Write tool descriptions like prompts.** The description is what the model sees. "Turn a named light on or off" beats "light controller function". Name the unfiltered state (`any`, `unspecified`) instead of hoping the model omits a slot — see [Design a tool-call prompt](https://docs.tryllengine.com/v0.9/how-to/design-a-tool-call-prompt.html).
1. **Validate on the client.** A small model will hallucinate argument names eventually. Treat `arguments_json` as untrusted input.
1. **Use low temperature.** Tool calls are a classification / structure task, not a creative one. `temperature=0` is often the right answer.

## Edges and pitfalls

- **Arguments arrive as the model wrote them.** `arguments_json` is the model's argument object as JSON text. A number or boolean stays whatever JSON the model emitted — which may be `3`, `"3"`, or something else entirely. Parse it, check the types, and coerce in your client.
- **The model can invent tools.** A `ToolCall` node will happily parse `{"tool": "nuke_from_orbit"}` if the model emits it. The client must check the tool name against the allow-list before acting.
- **Multiple calls per turn need `parallel_tool_calls = true`.** Most templates cap generation at one call per turn unless this flag is set; with it on (and where the model's format supports it — verified on Qwen and Granite) a single turn like "open all the doors" can produce one call per door instead of a single vague one. The node records one entry per parsed call on the turn and fires one `ToolCallNotification` per call either way.
- **No streaming.** Tool-call generation is non-streaming by design — the parser needs the full output. Do not expect `AnswerText` frames from a `ToolCall` node unless `mode = call_or_answer` and no tool was called.

## Related

- [Reference: ToolCall node](https://docs.tryllengine.com/v0.9/reference/nodes/tool-call.html)
- [Reference: Agent Parameters](https://docs.tryllengine.com/v0.9/reference/agent-parameters.html)
- [Reference: Wire Protocol — `ToolCallNotification`](https://docs.tryllengine.com/v0.9/reference/wire-protocol.html)
- [Concept: Workflows, Graphs, and Nodes](https://docs.tryllengine.com/v0.9/concepts/workflows-graphs-nodes.html)
- [Concept: Constrained output (GBNF)](https://docs.tryllengine.com/v0.9/concepts/constrained-output.html) — the lighter-weight cousin for a fixed `<action> <object>` shape when you don't need typed JSON args
- [How-to: Define and handle tool calls](https://docs.tryllengine.com/v0.9/how-to/define-and-handle-tool-calls.html)
- [How-to: Test tool calls in Chat and the Dialog Lab](https://docs.tryllengine.com/v0.9/how-to/test-tool-calls-in-editor.html)
- [How-to: Compare tool-call models](https://docs.tryllengine.com/v0.9/how-to/compare-tool-call-models.html)
- [How-to: Design a tool-call prompt](https://docs.tryllengine.com/v0.9/how-to/design-a-tool-call-prompt.html)
