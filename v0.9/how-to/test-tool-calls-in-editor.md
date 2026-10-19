# Test Tool Calls in Chat and the Dialog Lab

Exercise a [`ToolCall`](https://docs.tryllengine.com/v0.9/reference/nodes/tool-call.html) workflow in the Unity or Unreal editor without entering Play / PIE: talk to it in Chat, then script it in the Dialog Lab — either with canned results, or with the same live handler the game will run.

This page is about **editor testing**. To declare tools and handle them in game code, start with [Define and handle tool calls](https://docs.tryllengine.com/v0.9/how-to/define-and-handle-tool-calls.html).

Prerequisites

- A workflow with a `ToolCall` node (`disposition = AwaitResult` if the model should see the result).
- A language model whose chat template supports tool calling. See [Tool Calling](https://docs.tryllengine.com/v0.9/concepts/tool-calling.html).
- A scene `TryllAgentComponent` / `UTryllAgentComponent` with that workflow assigned.

______________________________________________________________________

## Register the handler on the scene component

Both Chat and the Lab reach tools through the **scene** agent component, not through a Play-mode instance. Register the handler on that component in edit mode, or the turn pauses with nobody to answer.

Put the handler on a sibling `MonoBehaviour` with `[ExecuteAlways]`, and register in `OnEnable` — the same pattern as the demo `GetPriceTool`:

```
[ExecuteAlways]
[RequireComponent(typeof(TryllAgentComponent))]
public sealed class GetPriceTool : MonoBehaviour
{
    private void OnEnable()
    {
        GetComponent<TryllAgentComponent>()
            .RegisterTool("get_price", HandleGetPrice);
    }

    private string HandleGetPrice(TryllToolCall call)
    {
        // Parse call.ArgumentsJson, return a string the model will see.
        return "450";
    }
}
```

Register in `OnRegister` (not `BeginPlay`) so Chat and the Lab see the handler outside PIE — the same pattern as `UTryllGetPriceTool`:

```
void UMyPriceTool::OnRegister()
{
    Super::OnRegister();
    Agent = GetOwner()->FindComponentByClass<UTryllAgentComponent>();
    if (Agent != nullptr)
    {
        Agent->RegisterTool(TEXT("get_price"),
            FTryllSimpleToolHandler(
                [this](const FTryllToolCall& Call) { return LookUp(Call); }));
    }
}
```

Chat's agent is not Play's agent. Unity Chat calls `CreateAgent` on the scene component, so handlers already registered there are applied automatically. Unreal Chat builds a separate preview agent and copies the component's registered handlers onto it when the preview attaches; later `RegisterTool` / `UnregisterTool` calls are forwarded to that preview as well.

______________________________________________________________________

## Chat — one conversation, inspect the call

1. Register the handler as above.
1. Open **Window → Tryll → Chat** (Unity) or **Window → Tryll → Tryll Chat** (Unreal). Assign the scene agent and press **Start**.
1. Send a line that should trigger the tool.
1. Select the assistant answer and open the [turn inspector](https://docs.tryllengine.com/v0.9/reference/turn-inspector.html). The **Tool calls** block shows the name, arguments, and result the handler returned.

If the turn stays paused, or ends as `PauseTimedOut` (a `ToolCall` with `pause_timeout_ms` `0` inherits the server's `workflow.tool_call_pause_timeout_ms`, 2000 ms in the shipped `server-config.json`), nothing registered that tool on the component Chat is using. Check that the handler's `OnEnable` / `OnRegister` ran (the GameObject / actor is enabled) and that the tool name matches the workflow exactly.

______________________________________________________________________

## Dialog Lab — canned results (default)

The Lab creates its **own** agent per variant. By default it does **not** call your scene handler.

Author a `ToolResults` entry on each script step that should call a tool (tool name + result string). Leave a step **without** an entry to assert that the model must not call that tool there. An unexpected call, a missing expected call, or a different tool than the one listed is a **CheckFail** on that cell — the same amber verdict as `MustContain` — and later steps still run. That assertion holds even if `GetPriceTool` is registered on the same merchant.

That is the default. No extra toggle.

See [Compare dialog variants](https://docs.tryllengine.com/v0.9/how-to/compare-dialog-variants.html) for scripts, repeats, and checks.

______________________________________________________________________

## Dialog Lab — live tools (opt-in)

Tick **Use live tools** on the variant when the point of the run is to exercise the real handler (a stateful price ladder, a quest flag, …).

- Canned results still win. Pin a step with `ToolResults` when you want a fixed value.
- An uncanned call is answered by the scene agent's `RegisterTool` handler, if it has one for that name.
- If the variant is opted in but the scene agent has no handler, the cell still fails — same message as the default.

Without the toggle, a live `GetPriceTool` on the merchant will **not** answer Lab calls. That is deliberate: existing scripts that left a step uncanned to prove the model stayed quiet keep that meaning.

The Lab snapshots the flag with the run (JSONL `use_live_tools`). The scene object itself is not part of that snapshot.

______________________________________________________________________

## Reset stateful handlers between Lab repeats

One Lab agent serves every repeat of a variant, and that agent is not the scene component's, so `OnAgentCreated` / `OnAgentReady` never fire for the Lab. A handler that keeps per-conversation state (call counts, ladders) must reset at the start of each repeat.

When **Use live tools** is on, the Lab dispatches a parameterless `ResetToolState` once per repeat, before any turn of it:

- **Unity:** `gameObject.SendMessage("ResetToolState")` on the scene agent's GameObject. Implement a public `void ResetToolState()` on a sibling `MonoBehaviour`.
- **Unreal:** any parameterless `UFUNCTION()` named `ResetToolState` on a component of the owning actor.

The name is the contract. Renaming it silently stops Lab repeats from resetting. A stateless handler declares nothing.

In game and in Chat the handler should also subscribe to the component's own create / interactions-removed events — those *do* fire there.

______________________________________________________________________

## Related

- [Define and handle tool calls](https://docs.tryllengine.com/v0.9/how-to/define-and-handle-tool-calls.html)
- [Test an agent in the editor](https://docs.tryllengine.com/v0.9/how-to/test-agent-in-editor.html)
- [Compare dialog variants in the Lab](https://docs.tryllengine.com/v0.9/how-to/compare-dialog-variants.html)
- [Concept: Tool calling](https://docs.tryllengine.com/v0.9/concepts/tool-calling.html)
- [Reference: ToolCall node](https://docs.tryllengine.com/v0.9/reference/nodes/tool-call.html)
- [Reference: Turn inspector](https://docs.tryllengine.com/v0.9/reference/turn-inspector.html)
