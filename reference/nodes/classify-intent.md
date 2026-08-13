# Classify Intent

The Classify Intent node maps a user message to a discrete **intent label** by performing top-1 vector similarity search over a small, labelled [embedded string storage](https://docs.tryllengine.com/reference/embedded-string-storage.html). When a label is found it attaches an `IntentionComponent` to the current [interaction](https://docs.tryllengine.com/reference/glossary.html#interaction) so a downstream [IntentToInstruction](https://docs.tryllengine.com/reference/nodes/intent-to-instruction.html) node can branch the graph or look up a matching instruction.

It does **not** generate text and does **not** render anything into the prompt — the `IntentionComponent` is routing-only; it is consumed by `IntentToInstructionNode` and not by `DefaultProjectionStrategy`.

`NodeType`: `ClassifyIntent`.

## How it works

1. Reads the `input`-resolved slot (empty = the `user_message` slot).
1. Embeds it with the embedding model recorded in the KB config file.
1. Calls `Search(queryVector, topK=1, predicate)` on the storage — using the same compiled-filter pipeline as [Retrieve](https://docs.tryllengine.com/reference/nodes/retrieve.html).
1. If the top hit's cosine distance is ≤ `threshold`, reads the configured `metadata_field` from that record's typed metadata.
1. Attaches `IntentionComponent{intentLabel}` to the interaction and returns `found`. Otherwise returns `not_found`.

The knowledge base records must carry a **string-typed** metadata field whose value is the intent label. See [KB schema requirements](#kb-schema-requirements).

## Parameters

| Param                     | Type          | Default               | Range      | Structural | Description                                                                                                                                                                                      |
| ------------------------- | ------------- | --------------------- | ---------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `embedded_string_storage` | Optional[str] | inherit model default | —          | ✓          | Named embedded string storage. Structural.                                                                                                                                                       |
| `input`                   | Optional[str] | inherit model default | —          | ✓          | Slot name this node embeds and searches with. Empty = "user_message". Structural: immutable after creation — rebinding would re-wire the slot dataflow that is validated once at agent creation. |
| `metadata_field`          | Optional[str] | inherit model default | —          | —          | Storage metadata field whose string value becomes the intent label.                                                                                                                              |
| `threshold`               | float         | 0.25                  | 0.0 – 1.0  | —          | Maximum cosine distance; hits above this are treated as not_found.                                                                                                                               |
| `filter`                  | Optional[str] | inherit model default | —          | —          | JSON filter applied during search. Empty = no filter. Mutable.                                                                                                                                   |
| `notify_client`           | bool          | False                 | —          | —          | When true, fire OnNodeEvent("intent_classified", …) on the found path.                                                                                                                           |
| `diagnostic_topk`         | int           | 5                     | 1.0 – 64.0 | —          | Hits to fetch for diagnostic purposes (routing always uses top-1).                                                                                                                               |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit        | Param field      | Description                                                                          |
| ----------- | ---------------- | ------------------------------------------------------------------------------------ |
| `found`     | `found_exit`     | Exit taken when classification produced a label above threshold. Empty string = END. |
| `not_found` | `not_found_exit` | Exit taken when no label cleared the threshold. Empty string = END.                  |

## Exit routes

| Route       | Fires when                                                                                                                                                       |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `found`     | Top-1 hit survived threshold filtering and its `metadata_field` contained a non-empty string. `IntentionComponent` is attached.                                  |
| `not_found` | Resolved `input` slot missing/empty, empty storage, all hits above threshold, or matched record's `metadata_field` is absent or empty. No component is attached. |

Both routes **must** be wired. An unwired exit is a graph compilation failure (error [`3003`](https://docs.tryllengine.com/reference/error-codes.html#3xxx-agent)).

## Side effects

- Embeds the `input`-resolved text (empty = `user_message`) using the embedding model recorded in the KB config file referenced by `embedded_string_storage`.
- Attaches an `IntentionComponent{intentLabel}` to the current interaction on `found`. The component is **not** rendered into the prompt by `DefaultProjectionStrategy` — it is consumed only by `IntentToInstructionNode`.
- On `not_found`, nothing is attached (a warning is logged if the top hit was rejected for a metadata reason rather than a distance reason).
- When `notify_client = "true"`, fires one `NodeEvent` (`event_type="intent_classified"`) on the `found` path, out-of-band, before the turn completes. Subscribe via `agent.set_on_intent_classified(cb)` (Python), `agent.SetOnIntentClassified(cb)` (C++), `client.IntentClassified` (Unity), or `UTryllSubsystem::OnIntentClassified` (Unreal). Events for unrecognised event_types fall through to the generic `OnNodeEvent` fallback on each client.

## KB schema requirements

The storage's `*.json` config must declare a `fields` array with at least one entry whose `type` is `"string"` — this is the field you reference in the `metadata_field` parameter:

```
{
  "version": 1,
  "embedding_model": "All-MiniLM-L6-v2 (Q4_K_M)",
  "records_file": "my_intents.kb.json",
  "index_file":   "my_intents.usearch",
  "fields": [
    { "name": "intent", "type": "string", "default": "" }
  ]
}
```

Each record in `my_intents.kb.json` carries the label in that field:

```
[
  { "id": "ex1", "text": "Move aside, you fool.", "metadata": { "intent": "insult" } },
  { "id": "ex2", "text": "May I please pass through?", "metadata": { "intent": "pleasing" } }
]
```

## Diagnostics

When `enable_diagnostics = true`, the node contributes these keys to `TurnComplete.debug_info`:

| Key                   | Meaning                                                                                    |
| --------------------- | ------------------------------------------------------------------------------------------ |
| `metadata_field`      | The configured metadata field name.                                                        |
| `input`               | The configured `input` slot name, when set (absent when using the `user_message` default). |
| `query`               | The `input`-resolved text that was embedded.                                               |
| `threshold`           | Max cosine distance used (or `"inf"` when disabled).                                       |
| `filter`              | Raw JSON of the active filter (empty when none).                                           |
| `raw_distance`        | Cosine distance of the top hit (empty on `not_found`).                                     |
| `result.intent`       | The intent label attached (empty on `not_found`).                                          |
| `result.record_index` | 0-based index of the matched record (empty on `not_found`).                                |
| `result.id`           | Record id of the matched example (empty on `not_found`).                                   |
| `result.text`         | Text of the matched example phrase (empty on `not_found`).                                 |

## Minimum working example

```
from tryll_client.graph import (
    GraphDescription, ClassifyIntentParams, IntentToInstructionParams,
    TransformParams, GenerateParams,
)

# 1. Create intent KB (file-backed, Path A)
client.create_embedded_string_storage(
    name="intent_kb",
    config_path="data/intents/intents.json",
)

# 2. Create instruction map (Map kind)
client.create_keyed_string_storage(
    name="intent_instructions",
    keys=["insult", "pleasing"],
    values=[
        "The user insulted you — respond curtly.",
        "The user is being polite — acknowledge grudgingly.",
    ],
)

# 3. Build the graph
graph = (
    GraphDescription()
    .add_node("classify", ClassifyIntentParams(
        embedded_string_storage="guard_intents/guard_intents.json",
        metadata_field="intent",
        threshold=0.6,
        found_exit="pick",
        not_found_exit="respond",
    ))
    .add_node("pick", IntentToInstructionParams(
        string_storage="intent_instructions",
        output_name="tone",
        default_exit="compose",
    ))
    .add_node("compose", TransformParams(
        template="{{#instructions}}[{{text}}]\n{{/instructions}}{{user_message}}",
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
cp.embedded_string_storage = "guard_intents/guard_intents.json";
cp.metadata_field          = "intent";
cp.threshold               = 0.6f;
cp.found_exit              = "pick";
cp.not_found_exit          = "respond";

IntentToInstructionParamsT pp;
pp.string_storage = "intent_instructions";
pp.output_name    = "tone";
pp.default_exit   = "compose";

TransformParamsT tp;
tp.template_    = "{{#instructions}}[{{text}}]\n{{/instructions}}{{user_message}}";
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
        EmbeddedStringStorage = "guard_intents/guard_intents.json",
        MetadataField         = "intent",
        Threshold             = 0.6f,
        FoundExit             = "pick",
        NotFoundExit          = "respond",
    })
    .AddIntentToInstruction("pick", new TryllIntentToInstructionParams
    {
        StringStorage = "intent_instructions",
        OutputName    = "tone",
        DefaultExit   = "compose",
    })
    .AddTransform("compose", new TryllTransformParams
    {
        Template    = "{{#instructions}}[{{text}}]\n{{/instructions}}{{user_message}}",
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
CP->EmbeddedStringStorage = TEXT("guard_intents/guard_intents.json");
CP->bOverrideMetadataField = true;
CP->MetadataField = TEXT("intent");
CP->Threshold  = 0.6f;
CP->FoundExit    = TEXT("pick");
CP->NotFoundExit = TEXT("respond");

UTryllIntentToInstructionParams* PP = UTryllNodeParamsFactory::MakeIntentToInstructionParams(this);
PP->bOverrideStringStorage = true;
PP->StringStorage = TEXT("intent_instructions");
PP->bOverrideOutputName = true;
PP->OutputName    = TEXT("tone");
PP->DefaultExit   = TEXT("compose");

UTryllTransformParams* TP = UTryllNodeParamsFactory::MakeTransformParams(this);
TP->bOverrideTemplate = true;
TP->Template    = TEXT("{{#instructions}}[{{text}}]\n{{/instructions}}{{user_message}}");
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

- **C++:** `GraphDescription::AddClassifyIntent(name, ClassifyIntentParamsT)` — `GraphDescription.h`
- **Python:** `GraphDescription.add_node(name, ClassifyIntentParams(...))` — `tryll_client.graph`
- **Unity:** `TryllGraphBuilder.AddClassifyIntent(name, new TryllClassifyIntentParams{...})` — `Runtime/Generated/TryllGraphBuilder.Nodes.cs`
- **Unreal:** `AddClassifyIntentNode(builder, name, UTryllClassifyIntentParams*)` — `Generated/TryllGraphBuilder.Nodes.h`
- **Unreal (Blueprint):** author `UTryllClassifyIntentParams` on a node description in a `UTryllWorkflowAsset`

## Related

- [Intent to Instruction](https://docs.tryllengine.com/reference/nodes/intent-to-instruction.html) — the natural downstream node.
- [Embedded String Storage](https://docs.tryllengine.com/reference/embedded-string-storage.html)
- [String Storage](https://docs.tryllengine.com/reference/string-storage.html) — for the `Map` kind used by `IntentToInstruction`.
- [How to build an intent-driven NPC](https://docs.tryllengine.com/how-to/build-intent-driven-npc.html)
- [Retrieve](https://docs.tryllengine.com/reference/nodes/retrieve.html) — similar architecture; top-K document retrieval.
- [Glossary](https://docs.tryllengine.com/reference/glossary.html#classify-intent-node)
