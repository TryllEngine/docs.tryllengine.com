# Cancel a Turn

Stop an agent's in-flight turn early — a chat **Stop** button, a barge-in, or a time-boxed decision that ran out of budget. You choose whether to **keep** the partial reply or **discard** the whole interaction as if it never happened.

Prerequisites

- An agent created and ready — see [Build a Chat Agent with a Graph](https://docs.tryllengine.com/v0.8/how-to/build-chat-agent-with-graph.html).
- A turn actually running (you typically call `Cancel` from your UI while `AnswerText` is streaming). Cancelling an **idle** agent is a no-op.

______________________________________________________________________

## How it works

`Cancel` sends a `CancelRequest` for the agent's current turn. The stop is **cooperative**: the node currently running finishes its step, then text generation stops at the next token and any TTS stops after the current audio chunk. The turn then ends normally through your **turn-complete** callback with status **`Cancelled`** — so the same handler that detects a finished turn also detects a cancelled one.

Two modes decide what happens to the turn's [interaction](https://docs.tryllengine.com/v0.8/reference/glossary.html#interaction):

| Mode                    | Effect                                                                                                     | Use for                                                                                                                    |
| ----------------------- | ---------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `StopAndKeep` (default) | Stop, but keep the partial reply in dialogue history.                                                      | A chat **Stop** button — the user sees what was generated so far and it stays in context.                                  |
| `StopAndDiscard`        | Stop **and** remove the last interaction (the user turn *and* the partial reply), as if it never happened. | A time-boxed decision agent that reached no decision, or a rollback where the half-finished turn must not pollute history. |

All four clients default to `StopAndKeep` (ordinal `0`).

Cancel from a different context than the blocking send

`Cancel` interrupts a turn that is already running, so it must come from a **different thread or callback** than the one blocked inside a synchronous send. In particular the Python `agent.send_message(...)` call blocks until the turn ends — call `agent.cancel()` from another thread. In the engines and the C++ callback model, send is non-blocking, so you cancel from the UI thread (or a streaming callback) while the turn streams.

______________________________________________________________________

## Stop button (`StopAndKeep`)

The common case: wire a **Stop** control to cancel the streaming turn, keeping what was generated.

```
using Tryll.Client;

var agentComp = GetComponent<TryllAgentComponent>();

// Hook a UI button. Default mode = StopAndKeep.
stopButton.onClick.AddListener(() => agentComp.Cancel());

// The turn ends through the same OnTurnComplete you already handle:
agentComp.OnTurnComplete.AddListener((status, _, _) =>
{
    if (status == TryllTurnStatus.Cancelled)
        Debug.Log("[Tryll] turn cancelled — kept partial reply");
});
```

```
// Call from your UI handler while the turn streams. Default = StopAndKeep.
AgentComponent->Cancel();

// React in your OnTurnComplete binding:
void AThisClass::HandleTurnComplete(ETryllTurnStatus Status,
                                    const FString& DebugInfo, int32 Tokens)
{
    if (Status == ETryllTurnStatus::Cancelled)
        UE_LOG(LogTemp, Log, TEXT("Turn cancelled — kept partial reply"));
}
```

1. On your Stop button, get the `TryllAgentComponent` and call **Cancel** (leave **Mode** at `StopAndKeep`).
1. Bind **On Turn Complete** and branch on `Status == Cancelled` to reset your "typing…" UI.

```
// Cancel is fire-and-forget (returns immediately). Call it from your UI/other
// thread while a turn streams; default mode = StopAndKeep.
agent.Cancel();   // ::Tryll::CancelMode_StopAndKeep

agent.SetOnTurnComplete([](::Tryll::TurnStatus status, std::string_view, std::int32_t)
{
    if (status == ::Tryll::TurnStatus_Cancelled)
        std::cout << "\n[cancelled]\n";
});
```

```
import threading
from tryll_client.graph import TurnStatus

# Observe the outcome via the turn-complete callback — it fires for a
# cancelled turn too.
agent.set_on_turn_complete(
    lambda status, debug_info, tokens:
        print("[cancelled] kept partial reply")
        if status == TurnStatus.Cancelled else None
)

# send_message blocks until the turn ends and returns the (partial) reply
# text, so run it off the main thread…
threading.Thread(
    target=lambda: agent.send_message("Tell me a very long story.")
).start()

# …then cancel from the main thread (e.g. on a keypress). Default = StopAndKeep.
agent.cancel()
```

`agent.cancel(mode=CancelMode.StopAndKeep, timeout=30.0)` blocks briefly waiting for the server `Ack`. The backgrounded `send_message` returns the partial reply that was streamed before the cancel.

______________________________________________________________________

## Rollback (`StopAndDiscard`)

When the half-finished turn must leave **no trace** — e.g. a decision-maker agent that timed out, or a speculative turn you're abandoning — discard it:

```
agentComp.Cancel(TryllCancelMode.StopAndDiscard);
```

```
AgentComponent->Cancel(ETryllCancelMode::StopAndDiscard);
```

```
agent.Cancel(::Tryll::CancelMode_StopAndDiscard);
```

```
from tryll_client.graph import CancelMode

agent.cancel(CancelMode.StopAndDiscard)
```

The user's message and the partial reply are both removed from the agent's dialogue, so the next turn sees history as though this turn never ran. (Any KV cache self-heals on the next send.)

______________________________________________________________________

## Notes

- **Cancel vs pause.** `Cancel` *ends* the turn; [Pause](https://docs.tryllengine.com/v0.8/how-to/pause-and-resume-turns.html) *suspends* it between nodes so you can mutate params and resume. `Cancel` is also the only way to abort a turn that is parked at a pause with no timeout.
- **Idle is a no-op.** Cancelling when no turn is running does nothing — safe to wire a Stop button that's always visible.
- **The editor chat windows** already do this: they swap **Send** for **Stop** (a `StopAndKeep` cancel) while a turn streams.

______________________________________________________________________

## Related

- [How to pause and resume a turn](https://docs.tryllengine.com/v0.8/how-to/pause-and-resume-turns.html)
- [How to stream answers to a UI](https://docs.tryllengine.com/v0.8/how-to/stream-answers-to-ui.html)
- [Reference: Wire Protocol — Cancelling a turn](https://docs.tryllengine.com/v0.8/reference/wire-protocol.html#cancelling-a-turn)
- [Reference: Blueprint Catalog — Cancel](https://docs.tryllengine.com/v0.8/reference/client-api/unreal/blueprint-catalog.html)
