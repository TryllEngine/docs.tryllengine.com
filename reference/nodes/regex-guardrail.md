# Regex Guardrail

The Regex Guardrail node matches text from an input slot against a list of regex patterns (one per string in a [string storage](https://docs.tryllengine.com/reference/string-storage.html)) and branches the [graph](https://docs.tryllengine.com/reference/glossary.html#graph) on the result. Typical use: short-circuit disallowed, off-topic, or jailbreak-style inputs to a [canned-response node](https://docs.tryllengine.com/reference/nodes/canned-response.html) before they reach a [Generate node](https://docs.tryllengine.com/reference/nodes/generate.html).

By default the node reads the reserved `user_message` slot. Set `input` to any upstream slot (for example a `Transform` or `Retrieve` output) to guardrail that text instead.

Patterns are compiled once at agent creation and matched with `std::regex_search` — i.e. a pattern hits if it matches **anywhere** in the text, case-insensitively.

`NodeType`: `RegexGuardrail`.

String storage kinds

`RegexGuardrailNode` accepts any string storage kind (`List`, `Map`, `Multimap`) and uses the **list view** — each value is treated as a regex pattern regardless of any key. Topic-tagged deflection (writing the matched pattern's category key onto the interaction for use by a downstream `CannedResponseNode`) is planned as future work.

## Parameters

| Param            | Type          | Default               | Range | Structural | Description                                                                                                                                                                                                                              |
| ---------------- | ------------- | --------------------- | ----- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `string_storage` | Optional[str] | inherit model default | —     | —          | Named session string storage. Mutable — rebinds the underlying storage.                                                                                                                                                                  |
| `inline_strings` | Optional[Any] | inherit model default | —     | ✓          | Inline pattern list. Structural because it materialises the underlying StringStorage at construction; rebinding goes through string_storage.                                                                                             |
| `input`          | Optional[str] | inherit model default | —     | ✓          | Slot name this node matches against. Empty = "user_message" (same default as every other input-bearing node). Structural: immutable after creation — rebinding would re-wire the slot dataflow that is validated once at agent creation. |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit            | Param field          | Description                                                                          |
| --------------- | -------------------- | ------------------------------------------------------------------------------------ |
| `triggered`     | `triggered_exit`     | Exit taken when the input slot text matches a guardrail pattern. Empty string = END. |
| `not_triggered` | `not_triggered_exit` | Exit taken when no pattern matches. Empty string = END.                              |

Source resolution

Priority order:

1. `string_storage` as a **relative file path** under `storage_data_folder` (loaded on demand at `CreateAgent` time).
1. `string_storage` as the **name** of an explicitly created session storage (via `CreateStringStorageRequest`).
1. `inline_strings` authored inline.
1. Server default file (`default_guardrail_patterns_path` in `server-config.json`).

Patterns are matched **case-insensitively**. Compilation errors at agent-creation time fail graph validation with error [`3003`](https://docs.tryllengine.com/reference/error-codes.html#3xxx-agent).

## Exit routes

| Route           | Fires when                                          |
| --------------- | --------------------------------------------------- |
| `triggered`     | At least one pattern matched the input slot text.   |
| `not_triggered` | No patterns matched (or the input slot is missing). |

Both routes must be wired.

## Side effects

None on the dialog. The node does not modify the current [interaction](https://docs.tryllengine.com/reference/glossary.html#interaction), does not emit `AnswerText` frames, and does not touch the KV cache. It is pure routing.

## Diagnostics

When `enable_diagnostics = true`, this node contributes to `debug_info.nodes[].diagnostics`. See [Turn Diagnostics JSON](https://docs.tryllengine.com/reference/turn-diagnostics-json.html) for the envelope.

| Key                         | Meaning                                                                                  | Absent when |
| --------------------------- | ---------------------------------------------------------------------------------------- | ----------- |
| `parameters.string_storage` | Name of the pattern storage that was matched against.                                    | —           |
| `parameters.pattern_count`  | How many patterns compiled and ran. A count of `0` means the guardrail could never fire. | —           |
| `output.triggered`          | Whether any pattern matched.                                                             | —           |

The node deliberately records **which** storage and **how many** patterns, but never the pattern that matched or the text it matched against. Use the node entry's `exit_route` (`triggered` / `not_triggered`) together with the input slot in the turn's `interaction` snapshot when you need to reconstruct a decision.

## Minimum working example

```
from tryll_client.graph import (
    GraphDescription, RegexGuardrailParams,
    CannedResponseParams, GenerateParams,
)

client.create_string_storage(
    name="jailbreak_patterns",
    strings=[
        r"ignore\s+previous\s+instructions?",
        r"you\s+are\s+DAN",
    ],
)

graph = (
    GraphDescription()
    .add_node("guard", RegexGuardrailParams(
        string_storage="jailbreak_patterns",
        triggered_exit="refuse",
        not_triggered_exit="answer",
    ))
    .add_node("refuse", CannedResponseParams(
        default_exit="",   # uses server default; empty = END
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

client.CreateStringStorage("jailbreak_patterns", {
    R"(ignore\s+previous\s+instructions?)",
    R"(you\s+are\s+DAN)",
});

RegexGuardrailParamsT guardP;
guardP.string_storage    = "jailbreak_patterns";
guardP.triggered_exit    = "refuse";
guardP.not_triggered_exit = "answer";

// refuseP and answerP use defaults (server default canned responses / END)
CannedResponseParamsT refuseP;
GenerateParamsT       answerP;

GraphDescription graph;
graph.AddRegexGuardrail("guard",  std::move(guardP))
     .AddCannedResponse("refuse",  std::move(refuseP))
     .AddGenerate("answer",        std::move(answerP))
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
    .AddCannedResponse("refuse", new TryllCannedResponseParams())
    .AddGenerate("answer",       new TryllGenerateParams())
    .SetStartNode("guard")
    .SetDefaultModelName("My Local Model")
    .Build();
```

```
#include "Generated/TryllGraphBuilder.Nodes.h"
#include "Generated/TryllNodeParamsFactory.h"

auto* Subsystem = GetGameInstance()->GetSubsystem<UTryllSubsystem>();
Subsystem->RequestCreateStringStorage(
    TEXT("jailbreak_patterns"),
    { TEXT(R"(ignore\s+previous\s+instructions?)"),
      TEXT(R"(you\s+are\s+DAN)") });

UTryllRegexGuardrailParams* GuardP = UTryllNodeParamsFactory::MakeRegexGuardrailParams(this);
GuardP->bOverrideStringStorage = true;
GuardP->StringStorage    = TEXT("jailbreak_patterns");
GuardP->TriggeredExit    = TEXT("refuse");
GuardP->NotTriggeredExit = TEXT("answer");

FTryllGraphDescription Graph = FTryllGraphBuilder()
    .AddNode(TEXT("guard"),  GuardP)
    .AddNode(TEXT("refuse"), UTryllNodeParamsFactory::MakeCannedResponseParams(this))
    .AddNode(TEXT("answer"), UTryllNodeParamsFactory::MakeGenerateParams(this))
    .SetStartNode(TEXT("guard"))
    .SetDefaultModelName(TEXT("My Local Model"))
    .Build();
```

See the full flow in [How to use canned responses and guardrails](https://docs.tryllengine.com/how-to/use-canned-responses-and-guardrails.html).

## Client bindings

- **C++:** `GraphDescription::AddRegexGuardrail(name, RegexGuardrailParamsT)` — `GraphDescription.h`
- **Python:** `GraphDescription.add_node(name, RegexGuardrailParams(...))` — `tryll_client.graph`
- **Unity:** `TryllGraphBuilder.AddRegexGuardrail(name, new TryllRegexGuardrailParams{...})` — `Runtime/Generated/TryllGraphBuilder.Nodes.cs`
- **Unreal:** `AddRegexGuardrailNode(builder, name, UTryllRegexGuardrailParams*)` — `Generated/TryllGraphBuilder.Nodes.h`

## Related

- [String Storage](https://docs.tryllengine.com/reference/string-storage.html)
- [Canned Response](https://docs.tryllengine.com/reference/nodes/canned-response.html) — the common downstream node on the `triggered` route.
- [How to use canned responses and guardrails](https://docs.tryllengine.com/how-to/use-canned-responses-and-guardrails.html)
- [Server Configuration — default file](https://docs.tryllengine.com/reference/server-configuration.html#node-defaults)
- [Glossary](https://docs.tryllengine.com/reference/glossary.html#guardrail)
