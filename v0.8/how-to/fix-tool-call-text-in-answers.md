# Fix Tool-Call Text in Answers

What to check when the player sees a tool call instead of dialogue. A chat bubble reads like this:

```
<|tool_call>call:get_price{items:<|"|>Sushi<|"|>,player_offer:<|"|><|"|>}<tool_call|>
```

That is the model's own tool-call syntax — Gemma 4 in this example; on Mistral-family models the reply starts with `[TOOL_CALLS]`. It usually comes with this server warning:

```
[warning] [answer] stripped unknown native TTS tag '<|tool_call>' from audio input; original text is unchanged
```

The name in brackets (`answer`) is the node that produced the text. The warning means the voice skipped the markup. The chat text still contains it, so the bubble shows it.

The text can come from three places, and each has a different fix. The diagnostics for the turn tell you which one you have.

Prerequisites

- The agent was created with `enable_diagnostics = true`. In the editor, open the turn in the [turn inspector](https://docs.tryllengine.com/v0.8/reference/turn-inspector.html); elsewhere, read `TurnComplete.debug_info` ([Turn Diagnostics JSON](https://docs.tryllengine.com/v0.8/reference/turn-diagnostics-json.html)).
- The graph has a [`ToolCall`](https://docs.tryllengine.com/v0.8/reference/nodes/tool-call.html) node, or a prompt that mentions a tool.

______________________________________________________________________

## 1. Find the node that wrote the text

In `debug_info.nodes[]`, find the entry whose `diagnostics.output.text` is the bubble. Its `_type` and a few fields beside it tell you which case this is:

| The text is on                             | And you also see                                                                                                                       | Case                                                                          |
| ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| A `GenerateNode` or `GenerateAndSpeakNode` | `engine.chat_turn.tool_count: 0`                                                                                                       | [A speech node wrote a call](#2-a-speech-node-wrote-a-call)                   |
| A `ToolCallNode`                           | `output.parse_fallback: true`, `output.parse_failure: malformed_call`, `parameters.mode: call_or_answer`, `exit_route: no_tool_called` | [The tool node sent an unparsed call](#3-the-tool-node-sent-an-unparsed-call) |
| Either, on this turn and earlier ones      | An `assistant` message in `input.prompt[]` whose `content` is the markup                                                               | [The markup is in history](#4-the-markup-is-in-history) — fix A or B first    |

______________________________________________________________________

## 2. A speech node wrote a call

This is the common case. A `Generate` or `GenerateAndSpeak` node has no tools and no grammar (`tool_count: 0`, `has_grammar: false`), so it cannot call anything. It wrote the call as dialogue because its prompt asked it to.

Two things are usually true together:

- **The speech prompt tells the model to call the tool.** A line such as "Call get_price with the requested items" belongs on the `ToolCall` node. The speech node also sees earlier turns' real calls in its history, rendered in the model's tool syntax, so it has a template to copy.
- **The turn skipped the tool node.** Look at the nodes before the speech node. Often a classifier exited `not_found`, or the graph took a branch that does not pass through the `ToolCall` node. The speech node then gets the full trade procedure and no tool result.

For a [`ClassifyIntentLLM`](https://docs.tryllengine.com/v0.8/reference/nodes/classify-intent-llm.html) gate, `output.classification` says why it missed:

| `not_found_reason` | Meaning                                           | Usual fix                                                                                                          |
| ------------------ | ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `below_threshold`  | The top label's `top_prob` was under `threshold`. | Two labels claim the same line — make the descriptions disagree. Then set `threshold` from real `top_prob` values. |
| `below_margin`     | Top and second label were closer than `margin`.   | The same overlap. Add "— not …" clauses rather than lowering `margin`.                                             |

A short "yes" is the usual culprit: "yes, show me prices" and "yes, I agree to that price" split the probability, so neither label clears the gate.

**Fix:**

1. **Take tool instructions out of the speech prompt.** Tell the speech node how to use a result that is already in the conversation, and what to say when there is none:

   ```
   If the conversation contains a price returned for the item, quote it exactly.
   If it does not, do not state a price — name the goods, or ask which item the
   player means.
   ```

1. **Give `not_found` its own speech node.** Wire the classifier's `not_found_exit` to a `Generate` / `GenerateAndSpeak` node whose prompt has no trade procedure and no tool name. A missed classification then produces a clarifying line instead of a call.

1. **Fix the labels** as in the table above. See [Design an intent-classifier prompt](https://docs.tryllengine.com/v0.8/how-to/design-an-intent-classifier-prompt.html).

A classifier has only `found` and `not_found` exits. It does not route by label. If only some intents need the tool (a price question does, "what do you have?" does not), put a classifier whose only job is "does this line need the tool?" directly in front of the `ToolCall` node. Set its `not_found_intent` to the "no" label, so those lines exit `not_found` and skip the tool.

______________________________________________________________________

## 3. The tool node sent an unparsed call

The `ToolCall` node ran, and the model tried to call the tool, but the output could not be parsed as a call. `output.parse_fallback: true` means exactly that: the parse failed, **no call was recorded**, and `output.text` holds the raw output. The node exits `no_tool_called`.

What happens to that raw text depends on `mode`:

| `mode`                     | Raw text                                                                        |
| -------------------------- | ------------------------------------------------------------------------------- |
| `call_or_answer` (default) | Sent to the client as the answer and stored in history. **This is the bubble.** |
| `detect_only`              | Not sent. The node exits `no_tool_called` silently.                             |
| `require_call`             | Not sent. The turn continues with no tool result.                               |

Two fields next to `parse_fallback` say why the parse failed:

- **`output.parse_failure`** — `malformed_call` when the output contains the model's tool-call marker but the call does not parse. That is this case. `no_call` means the model answered in prose instead, which is not a leak.
- **`output.parse_error`** — where the parser gave up: a byte offset into `output.text` and the text from there on. The offset is usually the start of the call that failed, so `unparsed from byte 0` means the first call is already broken.

Read the call that starts at that offset. Common signs:

- Two calls glued together while `parallel_tool_calls` is `false`.
- A string closed with ordinary quotes (`"""`) instead of the model's quote token (Gemma 4's `<|"|>`).
- A required argument missing, or values the prompt never mentioned.

**Fix:**

1. If a separate speech node answers after `no_tool_called`, set `mode = detect_only`, so a failed parse is never shown to the player.
1. Set `temperature = 0` in the `ToolCall` node's `sampling`, the recommended setting for tool calls. A node left at `temperature = 1` samples freely inside the call as well as around it.
1. Make the schema easier to fill: fewer arguments, enums for known values, and a real sentinel (`none`, `any`) instead of an empty string. See [Design a tool-call prompt](https://docs.tryllengine.com/v0.8/how-to/design-a-tool-call-prompt.html).

Without diagnostics, the server log carries the same reason after `[LlamaCppChatTurn] tool-call parse failed, returning raw content:`.

______________________________________________________________________

## 4. The markup is in history

Each leaked bubble is stored as an assistant message (unless the node's `history_role` is `None`). On later turns it appears in `input.prompt[]` as an `assistant` entry whose `content` is the markup and which has **no** `tool_calls` array:

```
{ "role": "assistant", "content": "<|tool_call>call:get_price{items:[<|\"|>Sushi<|\"|>]}<tool_call|>" }
```

A real historical call looks different — empty `content` and a `tool_calls` array:

```
{ "role": "assistant", "content": "",
  "tool_calls": [ { "id": "c00000002", "name": "get_price", "arguments_json": "{\"items\":\"Sushi\"}" } ] }
```

Every saved bubble shows the model again that tool syntax is an acceptable reply, so leaks get more frequent as the conversation goes on. After fixing the cause, remove the leaked turns — see [Seed and edit dialog history](https://docs.tryllengine.com/v0.8/how-to/seed-and-edit-dialog-history.html) — or start a fresh conversation to test.

______________________________________________________________________

## Do not filter the markup out

The [output filter](https://docs.tryllengine.com/v0.8/how-to/filter-llm-output-artifacts.html) has no rule for tool-call markup, and deleting the tags in your client only hides the symptom:

- The turn still has no tool result, so the player still gets no price.
- The server has already stored the reply with the markup, so the next turn's prompt still contains it.

Fix the prompt, the routing, or the tool node.

______________________________________________________________________

## Related

- [Tool Call node](https://docs.tryllengine.com/v0.8/reference/nodes/tool-call.html#diagnosing-a-call-that-never-happened) — when the tool node itself does not call
- [Concept: Tool calling](https://docs.tryllengine.com/v0.8/concepts/tool-calling.html) — `mode`, `require_call` and history projection
- [Classify Intent (LLM)](https://docs.tryllengine.com/v0.8/reference/nodes/classify-intent-llm.html) — `threshold`, `margin`, `not_found_intent`
- [Keep merchant prices deterministic](https://docs.tryllengine.com/v0.8/how-to/keep-merchant-prices-deterministic.html) — an alternative that needs no tool call for prices
- [Turn Diagnostics JSON](https://docs.tryllengine.com/v0.8/reference/turn-diagnostics-json.html) — every field used on this page
