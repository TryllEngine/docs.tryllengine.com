# Canned Response

The Canned Response node emits a **fixed** reply chosen from a [string storage](https://docs.tryllengine.com/v0.10/reference/string-storage.html) according to its `selection_strategy`, and ends its turn without invoking a [language model](https://docs.tryllengine.com/v0.10/reference/glossary.html#language-model). Use it for scripted answers, refusal lines, and branches where you do not want the cost or variability of a generation pass.

`NodeType`: `CannedResponse`. Changing `string_storage` to a name that cannot be loaded, or to empty, returns `3007 InvalidParamValue` and leaves the previous responses in force.

String storage kinds

`CannedResponseNode` accepts any string storage kind (`List`, `Map`, `Multimap`) and uses the **list view** — values are iterated by position. Key-aware selection (picking a response by a runtime intent key) is planned as future work.

For intent-driven response routing today, use [`IntentToInstructionNode`](https://docs.tryllengine.com/v0.10/reference/nodes/intent-to-instruction.html) to inject a tone directive and let the `Generate` node produce the response.

## Parameters

### Storage

| Param            | Type                  | Default    | Range | Structural | Localizable | Description                                                                                          |
| ---------------- | --------------------- | ---------- | ----- | ---------- | ----------- | ---------------------------------------------------------------------------------------------------- |
| `string_storage` | str                   | empty      | —     | —          | —           | File of responses, one per line. Used instead of Lines. Both empty = the server's default responses. |
| `lines`          | list[LocalizableText] | empty list | —     | ✓          | —           | Responses typed here; used when String Storage is empty.                                             |

### Behavior

| Param                | Type                            | Default                                    | Range | Structural | Localizable | Description                                                                                |
| -------------------- | ------------------------------- | ------------------------------------------ | ----- | ---------- | ----------- | ------------------------------------------------------------------------------------------ |
| `selection_strategy` | CannedResponseSelectionStrategy | CannedResponseSelectionStrategy.RoundRobin | —     | —          | —           | Which response comes next: Random, First (always the first), or RoundRobin (each in turn). |

### Output

| Param          | Type        | Default               | Range | Structural | Localizable | Description                                                                                                       |
| -------------- | ----------- | --------------------- | ----- | ---------- | ----------- | ----------------------------------------------------------------------------------------------------------------- |
| `output_name`  | str         | empty                 | —     | ✓          | —           | Name later nodes use to read the chosen response (in their Input, or as the slot. tag). Empty = this node's name. |
| `send`         | SendAnswer  | SendAnswer.Whole      | —     | —          | —           | How the response reaches the game: Whole (in one piece), Streamed, or None (kept for later nodes only).           |
| `history_role` | HistoryRole | HistoryRole.Assistant | —     | —          | —           | Whether later turns remember this response as an assistant message. None = forgotten after this turn.             |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit      | Param field    | Description                                           |
| --------- | -------------- | ----------------------------------------------------- |
| `default` | `default_exit` | Node to run after the response. Empty = end the turn. |

Source resolution

Priority order:

1. `string_storage` as a **relative file path** under the process storage root (loaded on demand at `CreateAgent` time).
1. `string_storage` as the **name** of an explicitly created session storage (via `CreateStringStorageRequest`).
1. `lines` authored inline (each entry's text; virtual / session-scoped).
1. Server default file (`default_canned_responses_path` in `server-config.json`).

## Exit routes

| Route     | Fires when                                                                                                                            |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| `default` | Always — the node never branches. Set `default_exit` to another node name to continue, or leave it empty to terminate the turn (END). |

## Side effects

- Keeps the chosen response in the current [turn](https://docs.tryllengine.com/v0.10/reference/glossary.html#interaction) as the assistant's reply unless `history_role = None`.
- Emits it according to `send`: with the default `Whole`, one `AnswerText` frame (`is_delta = false`, `is_final = true`) with the full response; with `None`, nothing reaches the client.

No model inference, no KV-cache work, no network I/O beyond the `AnswerText` frame itself.

## Diagnostics

When `enable_diagnostics = true`, this node contributes to `debug_info.nodes[].diagnostics`. See [Turn Diagnostics JSON](https://docs.tryllengine.com/v0.10/reference/turn-diagnostics-json.html) for the envelope.

No model runs, so the node emits a `parameters` section only.

| Key                             | Meaning                                                                                                           | Absent when |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ----------- |
| `parameters.string_storage`     | Name of the storage the response came from. Empty for a storage registered inline rather than by name.            | —           |
| `parameters.string_count`       | How many responses the storage holds.                                                                             | —           |
| `parameters.last_index`         | 0-based index of the response chosen this time. With `round_robin` this is the cursor that advances turn to turn. | —           |
| `parameters.selection_strategy` | `first`, `round_robin` or `random`.                                                                               | —           |

The text that was actually emitted is not repeated here — read it from the node's `SlotComponent` in the turn's `interaction` snapshot, or from the answer itself.

## Minimum working example

```
from tryll_client.graph import (
    GraphDescription, RegexGuardrailParams,
    CannedResponseParams, GenerateParams,
)

# 1. Create the storage first
client.create_string_storage(
    name="refusal_lines",
    strings=[
        "I can't help with that request.",
        "That's outside what I can assist with.",
    ],
)

# 2. Reference it from the node
graph = (
    GraphDescription()
    .add_node("guard", RegexGuardrailParams(
        string_storage="jailbreak_patterns",
        triggered_exit="refuse",
        not_triggered_exit="answer",
    ))
    .add_node("refuse", CannedResponseParams(
        string_storage="refusal_lines",
        default_exit="",   # empty = END
    ))
    .add_node("answer", GenerateParams(
        default_exit="",   # empty = END
    ))
    .set_start_node("guard")
    .set_default_model_name("My Local Model")
)

agent = client.create_agent(graph)
```

```
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

// 1. Create the storage first
client.CreateStringStorage("refusal_lines", {
    "I can't help with that request.",
    "That's outside what I can assist with.",
});

// 2. Reference it from the node
RegexGuardrailParamsT guardP;
guardP.string_storage    = "jailbreak_patterns";
guardP.triggered_exit    = "refuse";
guardP.not_triggered_exit = "answer";

CannedResponseParamsT refuseP;
refuseP.string_storage = "refusal_lines";
// refuseP.default_exit = ""; // empty = END (the default)

GenerateParamsT answerP;
// answerP.default_exit = ""; // empty = END (the default)

GraphDescription graph;
graph.AddRegexGuardrail("guard",  std::move(guardP))
     .AddCannedResponse("refuse",         std::move(refuseP))
     .AddGenerate("answer",               std::move(answerP))
     .SetStartNode("guard")
     .SetDefaultModelName("My Local Model");

auto agent = client.CreateAgent(graph);
```

```
using Tryll.Client;

var graph = new TryllGraphBuilder()
    .AddRegexGuardrail("guard", new TryllRegexGuardrailParams
    {
        StringStorage    = "jailbreak_patterns",
        TriggeredExit    = "refuse",
        NotTriggeredExit = "answer",
    })
    .AddCannedResponse("refuse", new TryllCannedResponseParams
    {
        StringStorage = "refusal_lines",
    })
    .AddGenerate("answer", new TryllGenerateParams())
    .SetStartNode("guard")
    .SetDefaultModelName("My Local Model")
    .Build();
```

```
#include "Generated/TryllGraphBuilder.Nodes.h"
#include "Generated/TryllNodeParamsFactory.h"

auto* Subsystem = GetGameInstance()->GetSubsystem<UTryllSubsystem>();
Subsystem->RequestCreateStringStorage(
    TEXT("refusal_lines"),
    { TEXT("I can't help with that request."),
      TEXT("That's outside what I can assist with.") });

UTryllRegexGuardrailParams* GuardP = UTryllNodeParamsFactory::MakeRegexGuardrailParams(this);
GuardP->bOverrideStringStorage = true;
GuardP->StringStorage    = TEXT("jailbreak_patterns");
GuardP->TriggeredExit    = TEXT("refuse");
GuardP->NotTriggeredExit = TEXT("answer");

UTryllCannedResponseParams* RefuseP = UTryllNodeParamsFactory::MakeCannedResponseParams(this);
RefuseP->bOverrideStringStorage = true;
RefuseP->StringStorage.Path = TEXT("refusal_lines");

UTryllGenerateParams* AnswerP = UTryllNodeParamsFactory::MakeGenerateParams(this);

FTryllGraphDescription Graph = FTryllGraphBuilder()
    .AddNode(TEXT("guard"),  GuardP)
    .AddNode(TEXT("refuse"), RefuseP)
    .AddNode(TEXT("answer"), AnswerP)
    .SetStartNode(TEXT("guard"))
    .SetDefaultModelName(TEXT("My Local Model"))
    .Build();
```

See the full flow in [How to use canned responses and guardrails](https://docs.tryllengine.com/v0.10/how-to/use-canned-responses-and-guardrails.html).

## Client bindings

- **C++:** `GraphDescription::AddCannedResponse(name, CannedResponseParamsT)` — `GraphDescription.h`
- **Python:** `GraphDescription.add_node(name, CannedResponseParams(...))` — `tryll_client.graph`
- **Unity:** `TryllGraphBuilder.AddCannedResponse(name, new TryllCannedResponseParams{...})` — `Runtime/Generated/TryllGraphBuilder.Nodes.cs`
- **Unreal:** `AddCannedResponseNode(builder, name, UTryllCannedResponseParams*)` — `Generated/TryllGraphBuilder.Nodes.h`

## Related

- [String Storage](https://docs.tryllengine.com/v0.10/reference/string-storage.html)
- [Regex Guardrail](https://docs.tryllengine.com/v0.10/reference/nodes/regex-guardrail.html) — the common upstream node.
- [How to use canned responses and guardrails](https://docs.tryllengine.com/v0.10/how-to/use-canned-responses-and-guardrails.html)
- [Server Configuration — default file](https://docs.tryllengine.com/v0.10/reference/server-configuration.html#node-defaults)
- [Glossary](https://docs.tryllengine.com/v0.10/reference/glossary.html#canned-response)
