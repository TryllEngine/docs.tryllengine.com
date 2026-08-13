# Seed and Edit Dialog History

Append scripted user/assistant turns to an agent's dialog **without running the workflow graph**, or trim whole interactions from the tail. Use this to bootstrap context, open with an NPC line, or retry a turn by removing the last exchange and sending again.

Prerequisites

- An agent created and **idle** — no turn running, not paused, and no KV-cache lifecycle operation in flight.
- Dialog mutations are **strict idle-only**: unlike [Change agent parameters at runtime](https://docs.tryllengine.com/how-to/change-agent-parameters.html), they are rejected while a turn is **paused** (`AgentBusy` 3004).

Each append item is one **interaction** (a user side, an assistant side, or both). Empty strings omit that side; both empty → the pair is skipped. The server returns how many interactions were actually applied — there is no `TurnComplete`.

______________________________________________________________________

## Seed prior conversation

Inject several exchanges so the model sees real chat-template history before the player's first live message:

```
using Tryll.Client;

var comp = GetComponent<TryllAgentComponent>();
comp.AppendInteractions(new List<TryllDialogInteraction>
{
    new() { UserMessage = "Who are you?", AssistantMessage = "I'm the town guide." },
    new() { UserMessage = "Where is the market?", AssistantMessage = "North of the fountain." },
});
comp.SendMessage("Is it open today?");
```

```
TArray<FTryllDialogInteraction> History;
{
    FTryllDialogInteraction A;
    A.UserMessage = TEXT("Who are you?");
    A.AssistantMessage = TEXT("I'm the town guide.");
    History.Add(A);
}
AgentComponent->AppendInteractions(History);
AgentComponent->SendMessage(TEXT("Where is the market?"));
```

```
agent.AppendInteractions({
    {"Who are you?", "I'm the town guide."},
    {"Where is the market?", "North of the fountain."},
});
agent.SendText("Is it open today?");
```

```
agent.append_interactions([
    ("Who are you?", "I'm the town guide."),
    ("Where is the market?", "North of the fountain."),
])
agent.send_message("Is it open today?")
```

______________________________________________________________________

## Assistant opener (no user line)

Start with the NPC speaking first — omit the user side:

```
comp.AppendInteractions(new List<TryllDialogInteraction>
{
    new() { AssistantMessage = "Welcome! What brings you to the tavern?" },
});
```

```
FTryllDialogInteraction Opener;
Opener.AssistantMessage = TEXT("Welcome! What brings you to the tavern?");
AgentComponent->AppendInteractions({ Opener });
```

```
agent.AppendInteractions({{"", "Welcome! What brings you to the tavern?"}});
```

```
agent.append_interactions([(None, "Welcome! What brings you to the tavern?")])
```

______________________________________________________________________

## Retry by remove-and-resend

Drop the last whole interaction (user + assistant), then send the user message again:

```
comp.RemoveInteractionsFromEnd(1);
comp.SendMessage("Could you say that differently?");
```

```
AgentComponent->RemoveInteractionsFromEnd(1);
AgentComponent->SendMessage(TEXT("Could you say that differently?"));
```

```
agent.RemoveInteractionsFromEnd(1);
agent.SendText("Could you say that differently?");
```

```
agent.remove_interactions_from_end(1)
agent.send_message("Could you say that differently?")
```

Removal is **saturating**: `count >= dialog size` clears all interactions; `count = 0` is a no-op.

______________________________________________________________________

## Remove several interactions at once

Trim the last N whole exchanges — e.g. rewind after a branching menu:

```
std::uint32_t removed = agent.RemoveInteractionsFromEnd(3);
```

```
removed = agent.remove_interactions_from_end(3)
```

Check the returned count to confirm how many interactions were removed.

______________________________________________________________________

## Stateless one-shot seed

Agents with `maintain_dialogue_history = false` still project appended history for the **next** `SendMessage`, then clear dialog after that turn completes. Use append to inject context for a single classification or routing call:

```
agent.append_interactions([("context", "prior fact")])
label = agent.send_message("Classify this utterance.")
# dialog cleared after TurnComplete — seed does not persist
```

______________________________________________________________________

## vs cancel discard

[Cancel a turn](https://docs.tryllengine.com/how-to/cancel-a-turn.html) with `StopAndDiscard` removes the **last in-flight** interaction mid-turn after a cooperative cancel. Dialog mutation operates on **idle** agents and edits completed or injected history explicitly. Use cancel discard for aborting generation; use tail remove for editing history before the next send.

Either mutation invalidates the agent's reusable KV prefix; removal also resets the token-budget projection window. The next send pays re-sync cost.

______________________________________________________________________

## Related

- [Reference: Wire Protocol — Dialog history mutation](https://docs.tryllengine.com/reference/wire-protocol.html#dialog-history-mutation)
- [How to cancel a turn](https://docs.tryllengine.com/how-to/cancel-a-turn.html)
- [Concept: Agents and sessions](https://docs.tryllengine.com/concepts/agents-and-sessions.html)
