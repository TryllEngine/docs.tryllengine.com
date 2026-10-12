# Stream Answers to a UI

Pipe each `AnswerText` chunk from the server into your application's view layer as it arrives — the standard "typewriter" chat experience.

Prerequisites

- An agent created and ready to receive messages — see [Build a Chat Agent with a Graph](https://docs.tryllengine.com/v0.8/how-to/build-chat-agent-with-graph.html).

A node with `send = Streamed` sends one `AnswerText` frame per token chunk. Every chunk is a delta (`is_delta = true`), and the node's last chunk is an empty terminator with `is_final = true` and `text = ""`. A node with `send = Whole` sends a single chunk with the full text, `is_delta = false`, and `is_final = true`. A final `TurnComplete` frame carries the turn's outcome (success / error / cancelled) and the total token count.

## Steps

On `TryllAgentComponent`, bind two events. All callbacks fire on the Unity **main thread** — you can update UGUI / UI Toolkit widgets directly without marshalling.

```
var agentComp = GetComponent<TryllAgentComponent>();

// OnAnswerText fires for every streamed chunk. The first argument,
// nodeName, is the name of the node that produced this chunk — ignore
// it for a single-answer agent (see "Multiple senders" below).
agentComp.OnAnswerText.AddListener((nodeName, text, isDelta, isFinal) =>
{
    chatBubble.AppendText(text);
    if (isFinal) chatBubble.HideCursor();
});

// OnTurnComplete fires once the turn ends.
agentComp.OnTurnComplete.AddListener((status, _, _) =>
{
    typingIndicator.SetActive(false);
    chatBubble.CommitReply();
});
```

Without a component — after `RequestCreateAgentAsync` — register the same callbacks on the handle itself. They also fire on the main thread. Pass `null` to clear; a later call replaces the previous one.

```
agent.SetOnAnswerText((nodeName, text, isDelta, isFinal) =>
{
    chatBubble.AppendText(text);
    if (isFinal) chatBubble.HideCursor();
});
agent.SetOnTurnComplete((status, _, _) =>
{
    typingIndicator.SetActive(false);
    chatBubble.CommitReply();
});
```

On the `UTryllAgentComponent`, bind two events in Blueprint:

- **On Answer Text (NodeName: FString, Text: FString, bIsFinal: bool)** — append `Text` to your `UTextBlock`. `bIsFinal` is `true` only on the node's last chunk (an empty terminator when the node streams). `NodeName` identifies the producing node — ignore it for a single-answer agent (see "Multiple senders" below).
- **On Turn Complete (Status: ETryllTurnStatus)** — stop the typing indicator and commit the final reply. Check `Status`: `Cancelled` means the user (or your Blueprint) called **Cancel**.

There is also **On Answer Full (FullText: FString)** which fires once after the turn completes, with the whole response in one string — useful if your UI only needs the final text.

To add a chat **Stop** button, call **Cancel** (`ETryllCancelMode::StopAndKeep` by default) while a turn is streaming, then gate your UI on `OnTurnComplete.Status == Cancelled`. Prefer `Status` over **On Answer Full** alone: after `StopAndDiscard` the server drops the interaction, but the component may still broadcast whatever text it had already accumulated client-side.

Register `SetOnAnswerText` before calling `SendText`. `SetOnAnswerText` fires on the client library's reader thread for each streamed chunk and once more with `isFinal = true` on the last chunk. `SendText` is fire-and-forget; use `SetOnTurnComplete` to know when the turn is finished.

```
std::string buffer;

agent.SetOnAnswerText(
    [&](std::string_view /*nodeName*/, std::string_view text,
        bool /*isDelta*/, bool isFinal)
    {
        buffer.append(text);
        YourChatWidget::SetCurrentReply(buffer);
        if (isFinal)
        {
            YourChatWidget::CommitReply(buffer);
            buffer.clear();
        }
    });

agent.SetOnTurnComplete(
    [](::Tryll::TurnStatus, std::string_view, std::int32_t) {});

agent.SendText("Tell me about the architecture.");
```

Register a persistent callback with `set_on_answer_text`. It fires on the client's background reader thread for every chunk with `(node_name, text, is_delta, is_final)`, while `send_message` blocks your calling thread until `TurnComplete` and returns the full reply.

```
def on_text(node_name, text, is_delta, is_final):
    your_chat_widget.append_text(text)   # node_name: see "Multiple senders"
    if is_final:
        your_chat_widget.hide_cursor()

agent.set_on_answer_text(on_text)

# Blocks until TurnComplete; on_text fires per chunk meanwhile.
reply = agent.send_message("Tell me about the architecture.")

# Diagnostics about the last turn:
print(agent.last_tokens_generated,
      agent.last_answer_chunk_count,
      agent.last_ttft_s)
```

The callback runs on the reader thread.

`on_text` fires on the client's background reader thread, not your calling thread — keep it quick and thread-safe, and don't call blocking client methods from inside it. If you only need the final text, skip the callback and use `send_message`'s return value.

## What the frames look like

| Frame          | Fires               | Payload                                 |
| -------------- | ------------------- | --------------------------------------- |
| `AnswerText`   | Many times per turn | `node_name`, `text` (delta), `is_final` |
| `TurnComplete` | Once at the end     | `status`, `tokens_generated`            |

`is_final` is `true` on each sending node's last `AnswerText`. For a streamed node that chunk is the empty terminator, so appending its text is harmless. It is normally safe to ignore and just rely on `TurnComplete` to lock in the reply; `is_final` is useful when you want to switch UI state slightly earlier (e.g., hide the cursor blink before the "turn done" animation).

## Streaming only part of the output

Nodes that are not `Generate` do not typically emit `AnswerText` frames. For example:

- `CannedResponse` emits **one** `AnswerText` with the full response and `is_final = true` — not a stream, but it uses the same callback.
- `ToolCall` with `mode = call_or_answer` emits the residual text the same way when no tool was called — one shot, `is_final = true`.
- `ToolCall` does *not* use `AnswerText` for the tool call itself; a non-`RouteOnly` `disposition` fires a separate `ToolCallNotification` (or a pausing disposition triggers `OnPaused`). See [Define and Handle Tool Calls](https://docs.tryllengine.com/v0.8/how-to/define-and-handle-tool-calls.html).

## Multiple senders in one turn

The first argument to every answer callback (`nodeName` / `node_name` / `NodeName`) is the name of the node that produced the chunk. For a single-answer agent you can ignore it. When a turn has more than one sending node — two characters speaking, or a "thinking" channel alongside the reply — switch on it to route each stream to its own widget. See [Send multiple answers in one turn](https://docs.tryllengine.com/v0.8/how-to/send-multiple-answers-in-a-turn.html).

## Common pitfalls

- **Waiting for `is_final`** as a synchronisation point is fine inside a `Generate` turn but misleading for `CannedResponse` — you will get one chunk with `is_final = true`. Always also bind `TurnComplete` for "turn is really done".
- **Threading.** `OnAnswerText` in Unity and Unreal is dispatched on the main/game thread; you can touch UI widgets directly. In C++, the `SendText` callback fires on the client library's reader thread — marshal to your UI thread explicitly. In Python, `set_on_answer_text` fires on the reader thread while `send_message` blocks the calling thread; marshal to your UI thread as needed.
- **Accumulating bytes, not chars.** For multi-byte scripts, concatenate the `FString` / `std::string` as you receive them; the server already chunks on UTF-8 boundaries.

## Related

- [Reference: Wire Protocol](https://docs.tryllengine.com/v0.8/reference/wire-protocol.html)
- [Reference: Workflow Nodes](https://docs.tryllengine.com/v0.8/reference/nodes/index.html)
- [Concept: Architecture at a Glance](https://docs.tryllengine.com/v0.8/concepts/architecture-at-a-glance.html)
- [How-to: Build a Chat Agent with a Graph](https://docs.tryllengine.com/v0.8/how-to/build-chat-agent-with-graph.html)
