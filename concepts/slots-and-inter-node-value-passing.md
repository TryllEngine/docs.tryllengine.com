# Slots and Inter-node Value Passing

Every producing node in a Tryll workflow — [`Generate`](https://docs.tryllengine.com/reference/nodes/generate.html), [`GenerateAndSpeak`](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html), [`CannedResponse`](https://docs.tryllengine.com/reference/nodes/canned-response.html), [`Transform`](https://docs.tryllengine.com/reference/nodes/transform.html), and the instruction nodes — writes its output into a **slot**: a named piece of text on a shared, per-turn blackboard. Downstream nodes in the *same* turn, and Mustache templates anywhere in the graph, can read any slot that has been written so far. This page explains the model; see the node reference pages for each node's specific `input`/`output_name`/`send`/`history_role` behaviour.

## The blackboard

Each turn has one blackboard (an `Interaction`). At the start of the turn the client's message is written into the reserved slot `user_message`. As the graph runs, every node that produces text writes it into a slot too — named by that node's `output_name` if set, otherwise the node's own name. Slot names must match `[A-Za-z_][A-Za-z0-9_]*` and `user_message` is reserved (no node may claim it).

## Reading a slot: `input`

Nodes that consume text — `Generate`, `GenerateAndSpeak`, `Retrieve`, `Speak`, [`ClassifyIntent`](https://docs.tryllengine.com/reference/nodes/classify-intent.html), [`ClassifyIntentLLM`](https://docs.tryllengine.com/reference/nodes/classify-intent-llm.html), and [`ToolCall`](https://docs.tryllengine.com/reference/nodes/tool-call.html) — have an `input` parameter: the name of the slot to read. Leave it empty for the default, `user_message`, uniformly across every one of these nodes (including `Speak`, whose empty `input` used to auto-voice the latest assistant-role slot instead — that special-cased default was removed in favor of this uniform rule). Set it to any upstream node's name to consume that node's output instead — this is how you chain nodes: a `Transform` (or `Generate`) rewrites or composes text into its own slot, and the next node reads it via `input`.

```
flowchart LR
    rewrite["Transform<br>rewrite"] -->|input=rewrite| retrieve["Retrieve<br>retrieve"]
    retrieve --> generate["Generate<br>generate"]
```

`ClassifyIntentLLM` is a partial exception: `input` only re-selects the **current** turn's user-area message; the `history_turns` of past turns it replays for coreference always use the plain `user_message` slot, matching `Generate`'s own history projection.

## Reading a slot in a template: `{{slot.<name>}}`

Any Mustache `template` (on `Generate`, `GenerateAndSpeak`, or `Transform`) can reference a slot directly with `{{slot.<name>}}` — this is a *look-up*, distinct from `input` (which *replaces* the resolved user turn). Use `{{slot.<name>}}` to weave a slot's value into a larger composed message without displacing the conversational turn structure. `{{user_message}}` is always available as the raw client input regardless of `input`. See [Use Mustache templates](https://docs.tryllengine.com/how-to/use-mustache-templates.html) for the full tag reference (`{{#instructions}}`, `{{knowledge_<source>}}`, …).

## Controlling the wire: `send`

`send: SendAnswer` controls whether a node's text is streamed to the client as `AnswerText` frames:

| Value      | Behaviour                                                                                                                                |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `None`     | Captured into the slot only — never sent. Use this for internal/scratch nodes (a query-rewrite step, a draft the user should never see). |
| `Whole`    | One `AnswerText` frame, sent once the full text is ready.                                                                                |
| `Streamed` | Per-token/delta `AnswerText` frames as text is generated.                                                                                |

Every `AnswerText` frame carries `node_name`, so a client that cares which node produced a chunk of text (e.g. attributing multiple NPC speakers in one turn) can distinguish them; clients that don't care can ignore the field.

## Controlling replay: `history_role`

`history_role: HistoryRole` controls whether a slot's text is replayed as a separate assistant message in the projected prompt — both on *later* turns and, for downstream nodes, within the *same* turn it was written (see [Same-turn effects](#same-turn-effects) below):

| Value       | Behaviour                                                                                                                                                           |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `None`      | Never replayed — a scratch value that only exists within the turn it was written, readable only via `input`/`{{slot.<name>}}`.                                      |
| `Assistant` | Replayed as a separate assistant message, in the order it was produced, both to downstream nodes later in the same turn and on every later turn's projected prompt. |

`User` and `System` are reserved for future use and not implemented yet.

## Same-turn effects

Within the **same turn**, `send`/`history_role` have two independent effects on downstream nodes — neither is gated by "did this go out on the wire":

1. **Every slot is directly readable**, no matter its `send`/`history_role`. Any downstream node's `input` selector and any template's `{{slot.<name>}}` reference can see it. A `Transform` node with `send=None` can still feed a `Retrieve` or `Generate` node two steps later; nothing about "not on the wire" means "not available internally".
1. **`history_role=Assistant` slots additionally replay as assistant-role messages** into every downstream node's projected prompt within the same turn — the same replay mechanism used for later-turn history, applied within-turn to upstream outputs. A "draft → refine" chain gets the draft in context for free: the refine node sees it both as `{{slot.draft}}` and as a natural assistant message immediately before its own prompt.

## Naming and renaming

Slot names are just node names (or `output_name` overrides) — there's no separate GUID layer. Both the Unity and Unreal editors provide a rename commit-point for a node's name and for its `output_name` override: committing a new value there (Enter/focus-loss in Unity's delayed text fields, `OnTextCommitted` in Unreal's Details-panel customizations) automatically rewrites every exit field and `StartNode` referencing the old node name, every `input` selector naming the old effective slot, and every `{{slot.<old_name>}}` tag in every node's template/instruction text across the graph — as a single undoable transaction. The commit point also enforces the `[A-Za-z_][A-Za-z0-9_]*` grammar, rejects the reserved `user_message`, and uniquifies against existing node/slot names. The `input` field itself renders as a dropdown of only the slots that are may-reachable upstream of that node (`"User Message"` plus one entry per upstream producer) — no free-text entry — so stale references are visually distinguishable and structurally harder to create in the first place.

## Slots vs. Variables

Slots and [Agent Variables](https://docs.tryllengine.com/concepts/agent-variables.html) look similar (both render via a Mustache `{{namespace.<name>}}` tag) but serve opposite purposes: a slot is **per-turn and node-produced** — it exists only because a node ran this turn, and disappears with the turn unless replayed as history. A variable is **persistent and game-produced** — declared once at `CreateAgent`, it survives across every turn until the game changes it, and no node ever writes one. Use a slot to pass a node's output to another node or template within (or across) a turn; use a variable to inject external game state (player level, mood, inventory) into a prompt or a `Retrieve` filter without a `ChangeParam` round-trip.

## See also

- [Transform node](https://docs.tryllengine.com/reference/nodes/transform.html)
- [Agent Variables](https://docs.tryllengine.com/concepts/agent-variables.html)
- [How-to: query rewriting for RAG](https://docs.tryllengine.com/how-to/query-rewriting-for-rag.html)
- [Use Mustache templates](https://docs.tryllengine.com/how-to/use-mustache-templates.html)
- [Workflows, graphs, and nodes](https://docs.tryllengine.com/concepts/workflows-graphs-nodes.html)
