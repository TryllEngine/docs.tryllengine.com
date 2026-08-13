# Intent to Instruction

The Intent to Instruction node looks up the **intent label** written by a preceding [ClassifyIntent](https://docs.tryllengine.com/reference/nodes/classify-intent.html) node in a [`Map`-kind string storage](https://docs.tryllengine.com/reference/string-storage.html), and attaches the matching value as a `kind=Instruction` [slot](https://docs.tryllengine.com/concepts/slots-and-inter-node-value-passing.html) to the current [interaction](https://docs.tryllengine.com/reference/glossary.html#interaction).

The attached instruction is then rendered into the prompt by the downstream [Generate](https://docs.tryllengine.com/reference/nodes/generate.html) node's [Mustache template](https://docs.tryllengine.com/how-to/use-mustache-templates.html) via the `{{#instructions}}` loop or the `{{slot.<name>}}` direct lookup.

`NodeType`: `IntentToInstruction`.

## How it works

1. Reads `IntentionComponent` from the current interaction (written by an upstream `ClassifyIntent` node).
1. Calls `FindFirst(intentLabel)` on the configured `Map` string storage.
1. If a match is found, attaches `SlotComponent{outputName, matchedText, SlotKind::Instruction, ...}` and returns `"default"`.
1. If no `IntentionComponent` is present, or the key is not found in the map, logs a warning and returns `"default"` without attaching any component. The downstream `Generate` node's template renders the instructions loop as empty, producing a general answer.

The node **always** takes the `"default"` exit — it never blocks or branches. Missing intents and missing keys are handled silently (with a log warning) so the graph keeps running without hard failures.

## Parameters

| Param            | Type          | Default               | Range | Structural | Description                                                                                                                                                                                                                                     |
| ---------------- | ------------- | --------------------- | ----- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `string_storage` | Optional[str] | inherit model default | —     | —          | Named session Map-kind string storage. Mutable — rebinds the storage.                                                                                                                                                                           |
| `inline_keys`    | Optional[Any] | inherit model default | —     | ✓          | Inline map keys (equal-length with inline_strings). Structural because they materialise the in-memory Map storage at construction.                                                                                                              |
| `inline_strings` | Optional[Any] | inherit model default | —     | ✓          | Inline map values (equal-length with inline_keys). Structural.                                                                                                                                                                                  |
| `output_name`    | Optional[str] | inherit model default | —     | ✓          | Name this node's output slot is stored under. Empty = the node's own name. Downstream templates look it up via slot.. Structural: immutable after creation — renaming would re-wire the slot dataflow that is validated once at agent creation. |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit      | Param field    | Description                                                                        |
| --------- | -------------- | ---------------------------------------------------------------------------------- |
| `default` | `default_exit` | Default exit target after mapping an intent to an instruction. Empty string = END. |

## Exit routes

| Route     | Fires when                                                                                                                                  |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `default` | Always — the node never branches. Set `default_exit` to the downstream `Generate` node name, or leave it empty to terminate the turn (END). |

The `"default"` exit fires regardless of whether an instruction was found. When nothing is found the template's `{{#instructions}}` loop renders as empty.

## Side effects

- Attaches a `kind=Instruction` `SlotComponent{name=outputName, text}` to the current interaction when a matching key is found. The slot is rendered into the prompt by the downstream `GenerateNode` via:
  - `{{#instructions}}{{text}}{{/instructions}}` — iterates all attached instruction-kind slots.
  - `{{slot.<name>}}` — renders only the slot with the matching name.
- When no instruction is attached (missing `IntentionComponent` or missing key), `{{#instructions}}` renders as empty and the model produces a general response.

## Diagnostics

When `enable_diagnostics = true`, the node contributes these keys to `TurnComplete.debug_info`:

| Key              | Meaning                                                                                |
| ---------------- | -------------------------------------------------------------------------------------- |
| `string_storage` | The name of the map storage used.                                                      |
| `output_name`    | The name given to the emitted component.                                               |
| `intent`         | The intent label read from `IntentionComponent` (empty if no component was present).   |
| `found`          | `"true"` when the key was matched and the component was attached; `"false"` otherwise. |
| `result_text`    | The instruction text that was attached (empty when `found = "false"`).                 |

## Minimum working example

```
from tryll_client.graph import (
    GraphDescription, ClassifyIntentParams, IntentToInstructionParams,
    TransformParams, GenerateParams,
)

# Create a Map storage: key = intent label, value = instruction text
client.create_keyed_string_storage(
    name="npc_instructions",
    keys=["insult", "pleasing", "joking"],
    values=[
        "The user insulted you. Respond curtly and insult them back.",
        "The user is being polite. Show grudging satisfaction.",
        'The user told a joke. Laugh once ("Ha."), then refocus on duty.',
    ],
)

# The IntentToInstruction node is wired after a ClassifyIntent node
graph = (
    GraphDescription()
    .add_node("classify", ClassifyIntentParams(
        embedded_string_storage="intent_kb",
        embedding_model="All-MiniLM-L6-v2 (Q4_K_M)",
        metadata_field="intent",
        threshold=0.6,
        found_exit="pick",
        not_found_exit="respond",
    ))
    .add_node("pick", IntentToInstructionParams(
        string_storage="npc_instructions",
        output_name="tone",
        default_exit="compose",
    ))
    .add_node("compose", TransformParams(
        template="{{#instructions}}[Tone: {{text}}]\n{{/instructions}}{{user_message}}",
        default_exit="respond",
    ))
    .add_node("respond", GenerateParams(
        input="compose",
        default_exit="",   # empty = END
    ))
    .set_start_node("classify")
    .set_default_model_name("My Local Model")
)

agent = client.create_agent(graph)
```

```
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

ClassifyIntentParamsT cp;
cp.embedded_string_storage = "intent_kb";
cp.embedding_model         = "All-MiniLM-L6-v2 (Q4_K_M)";
cp.metadata_field          = "intent";
cp.threshold               = 0.6f;
cp.found_exit              = "pick";
cp.not_found_exit          = "respond";

IntentToInstructionParamsT pp;
pp.string_storage = "npc_instructions";
pp.output_name    = "tone";
pp.default_exit   = "compose";

TransformParamsT tp;
tp.template_    = "{{#instructions}}[Tone: {{text}}]\n{{/instructions}}{{user_message}}";
tp.default_exit = "respond";

GenerateParamsT gp;
gp.input = "compose";
// gp.default_exit = ""; // empty = END (the default)

GraphDescription graph;
graph.AddClassifyIntent("classify", std::move(cp))
     .AddIntentToInstruction("pick",     std::move(pp))
     .AddTransform("compose",            std::move(tp))
     .AddGenerate("respond",             std::move(gp))
     .SetStartNode("classify")
     .SetDefaultModelName("My Local Model");

auto agent = client.CreateAgent(graph);
```

```
using Tryll.Client;

var graph = new TryllGraphBuilder()
    .AddClassifyIntent("classify", new TryllClassifyIntentParams
    {
        EmbeddedStringStorage = "intent_kb",
        EmbeddingModel        = "All-MiniLM-L6-v2 (Q4_K_M)",
        MetadataField         = "intent",
        Threshold             = 0.6f,
        FoundExit             = "pick",
        NotFoundExit          = "respond",
    })
    .AddIntentToInstruction("pick", new TryllIntentToInstructionParams
    {
        StringStorage = "npc_instructions",
        OutputName    = "tone",
        DefaultExit   = "compose",
    })
    .AddTransform("compose", new TryllTransformParams
    {
        Template    = "{{#instructions}}[Tone: {{text}}]\n{{/instructions}}{{user_message}}",
        DefaultExit = "respond",
    })
    .AddGenerate("respond", new TryllGenerateParams
    {
        Input = "compose",
    })
    .SetStartNode("classify")
    .SetDefaultModelName("My Local Model")
    .Build();
```

```
#include "Generated/TryllGraphBuilder.Nodes.h"
#include "Generated/TryllNodeParamsFactory.h"

UTryllClassifyIntentParams* CP = UTryllNodeParamsFactory::MakeClassifyIntentParams(this);
CP->bOverrideEmbeddedStringStorage = true;
CP->EmbeddedStringStorage = TEXT("intent_kb");
CP->bOverrideEmbeddingModel = true;
CP->EmbeddingModel = TEXT("All-MiniLM-L6-v2 (Q4_K_M)");
CP->bOverrideMetadataField = true;
CP->MetadataField = TEXT("intent");
CP->Threshold  = 0.6f;
CP->FoundExit    = TEXT("pick");
CP->NotFoundExit = TEXT("respond");

UTryllIntentToInstructionParams* PP = UTryllNodeParamsFactory::MakeIntentToInstructionParams(this);
PP->bOverrideStringStorage = true;
PP->StringStorage = TEXT("npc_instructions");
PP->bOverrideOutputName = true;
PP->OutputName    = TEXT("tone");
PP->DefaultExit   = TEXT("compose");

UTryllTransformParams* TP = UTryllNodeParamsFactory::MakeTransformParams(this);
TP->bOverrideTemplate = true;
TP->Template    = TEXT("{{#instructions}}[Tone: {{text}}]\n{{/instructions}}{{user_message}}");
TP->DefaultExit = TEXT("respond");

UTryllGenerateParams* GP = UTryllNodeParamsFactory::MakeGenerateParams(this);
GP->bOverrideInput = true;
GP->Input = TEXT("compose");

FTryllGraphDescription Graph = FTryllGraphBuilder()
    .AddNode(TEXT("classify"), CP)
    .AddNode(TEXT("pick"),     PP)
    .AddNode(TEXT("compose"),  TP)
    .AddNode(TEXT("respond"),  GP)
    .SetStartNode(TEXT("classify"))
    .SetDefaultModelName(TEXT("My Local Model"))
    .Build();
```

See the end-to-end walkthrough in [How to build an intent-driven NPC](https://docs.tryllengine.com/how-to/build-intent-driven-npc.html).

## Client bindings

- **C++:** `GraphDescription::AddIntentToInstruction(name, IntentToInstructionParamsT)` — `GraphDescription.h`
- **Python:** `GraphDescription.add_node(name, IntentToInstructionParams(...))` — `tryll_client.graph`
- **Unity:** `TryllGraphBuilder.AddIntentToInstruction(name, new TryllIntentToInstructionParams{...})` — `Runtime/Generated/TryllGraphBuilder.Nodes.cs`
- **Unreal:** `AddIntentToInstructionNode(builder, name, UTryllIntentToInstructionParams*)` — `Generated/TryllGraphBuilder.Nodes.h`
- **Unreal (Blueprint):** author `UTryllIntentToInstructionParams` on a node description in a `UTryllWorkflowAsset`

## Related

- [Classify Intent](https://docs.tryllengine.com/reference/nodes/classify-intent.html) — the upstream node that writes the `IntentionComponent`.
- [String Storage](https://docs.tryllengine.com/reference/string-storage.html) — Map kind required.
- [Instruction](https://docs.tryllengine.com/reference/nodes/instruction.html) — static (hardcoded) alternative when you do not need runtime key lookup.
- [How to build an intent-driven NPC](https://docs.tryllengine.com/how-to/build-intent-driven-npc.html)
- [How to use Mustache templates](https://docs.tryllengine.com/how-to/use-mustache-templates.html)
- [Glossary](https://docs.tryllengine.com/reference/glossary.html#intent-to-instruction-node)
