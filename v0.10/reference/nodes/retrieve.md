# Retrieve

The Retrieve node performs the **R** of [RAG](https://docs.tryllengine.com/v0.10/reference/glossary.html#rag): vector similarity search over an [embedded string storage](https://docs.tryllengine.com/v0.10/reference/embedded-string-storage.html), attaching the top matches to the current [interaction](https://docs.tryllengine.com/v0.10/reference/glossary.html#interaction) as a [knowledge component](https://docs.tryllengine.com/v0.10/reference/glossary.html#knowledge-component). A `filter` that does not compile, changed via `ChangeAgentParam`, returns `3007 InvalidParamValue` and leaves the previous filter in force.

It does **not** modify the prompt directly. Rendering is delegated to the downstream `Generate` node's [Mustache template](https://docs.tryllengine.com/v0.10/how-to/use-mustache-templates.html), which decides where and how the attached components appear in the [prompt](https://docs.tryllengine.com/v0.10/reference/glossary.html#prompt).

`NodeType`: `Retrieve`.

Watch it built

- [Aquarium Expert](https://docs.tryllengine.com/v0.10/reference/video-walkthroughs.html#aquarium-expert) (9 min): Covers Input, Retrieval Mode, Top K, Threshold and the two exits.
- [Aquarium Expert, advanced path](https://docs.tryllengine.com/v0.10/reference/video-walkthroughs.html#aquarium-expert-advanced) (13 min): Switches to Hybrid and reads a hybrid hit table.

## Parameters

### Input

| Param   | Type | Default | Range | Structural | Localizable | Description                                                                                          |
| ------- | ---- | ------- | ----- | ---------- | ----------- | ---------------------------------------------------------------------------------------------------- |
| `input` | str  | empty   | —     | ✓          | —           | Text to search with: the player's message or an earlier node's output. Empty = the player's message. |

### Storage

| Param                     | Type                | Default                      | Range | Structural | Localizable | Description                                                                                                     |
| ------------------------- | ------------------- | ---------------------------- | ----- | ---------- | ----------- | --------------------------------------------------------------------------------------------------------------- |
| `source_kind`             | KnowledgeSourceKind | KnowledgeSourceKind.FromFile | —     | ✓          | —           | From File = a prebuilt knowledge file. From Name = a memory store the game fills during play (Write To Memory). |
| `embedded_string_storage` | str                 | empty                        | —     | ✓          | —           | Embedded knowledge file to search. Used when Source Kind is From File; leave empty with From Name.              |
| `filter`                  | str                 | empty                        | —     | —          | —           | JSON filter on chunk metadata; only matching chunks are searched. Empty = no filter.                            |
| `storage_name`            | str                 | empty                        | —     | ✓          | —           | Memory store to search, by name. Used when Source Kind is From Name. May include agent variables (var.\*).      |
| `schema`                  | str                 | empty                        | —     | ✓          | —           | Record schema of the store. Required with From Name; with From File, checked against the file.                  |

### Behavior

| Param            | Type          | Default             | Range        | Structural | Localizable | Description                                                                                                    |
| ---------------- | ------------- | ------------------- | ------------ | ---------- | ----------- | -------------------------------------------------------------------------------------------------------------- |
| `retrieval_mode` | RetrievalMode | RetrievalMode.Dense | —            | —          | —           | Dense = match by meaning. Lexical = match by keywords (BM25). Hybrid = both, merged.                           |
| `top_k`          | int           | 2                   | 1.0 – 1000.0 | —          | —           | How many matching chunks to return.                                                                            |
| `threshold`      | float         | 0.5                 | 0.0 – 2.0    | —          | —           | Max cosine distance for a chunk to count (0 = identical, 2 = opposite). 0 = no limit. Ignored in Lexical mode. |
| `rrf_k`          | int           | 60                  | 1.0 – 1000.0 | —          | —           | Hybrid only: how strongly the merge favours top-ranked hits. Higher = flatter. 60 is the standard value.       |

### Output

| Param    | Type | Default | Range | Structural | Localizable | Description                                                                          |
| -------- | ---- | ------- | ----- | ---------- | ----------- | ------------------------------------------------------------------------------------ |
| `source` | str  | empty   | —     | —          | —           | Name templates use for these results: the knowledge\_ tag. Empty = this node's name. |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit        | Param field      | Description                                                         |
| ----------- | ---------------- | ------------------------------------------------------------------- |
| `found`     | `found_exit`     | Node to run when at least one chunk is found. Empty = end the turn. |
| `not_found` | `not_found_exit` | Node to run when nothing is found. Empty = end the turn.            |

## Schema picker (Unity)

`schema` is a catalog name, not a free-form path. In the Unity Inspector the `[TryllSchema]` drawer reads the selected bundled server's `data/schemas.json`, offers `(none)`, and keeps an unknown value as **Unregistered**. An external server may expose a different catalog — the server remains authoritative. Keep both address fields visible: File mode still uses `embedded_string_storage`; Name mode uses `storage_name` + `schema`.

## Exit routes

| Route       | Fires when                                                             |
| ----------- | ---------------------------------------------------------------------- |
| `found`     | At least one chunk survived threshold filtering.                       |
| `not_found` | The resolved `input` text is empty, or zero chunks survived filtering. |

Each route can point at a different target or at the same target. An empty exit field routes to END and is valid. A non-empty exit that does not name a node in the graph fails agent creation with [`3003 GraphCompilationFailed`](https://docs.tryllengine.com/v0.10/reference/error-codes.html#3xxx-agent).

## Recommended Hybrid defaults

Default `retrieval_mode` remains **Dense** (no silent behaviour change). To opt in to hybrid retrieval:

| Param            | Suggested value                                                          |
| ---------------- | ------------------------------------------------------------------------ |
| `retrieval_mode` | `Hybrid`                                                                 |
| `top_k`          | `3`                                                                      |
| `threshold`      | `0.5` (dense-leg gate only — BM25-only hits are not filtered by it)      |
| `rrf_k`          | `60` (Cormack default; `10` scored slightly higher on the PZ eval sweep) |

**Release note:** switching an existing graph to Hybrid changes how `threshold` behaves — it no longer acts as a post-fusion precision guard. Prefer `top_k` for result-count control in Hybrid.

## Side effects

- In **Dense** / **Hybrid** modes, embeds the resolved `input` text (an empty `input` means the player's message) using the embedding model recorded in the KB config file referenced by `embedded_string_storage`. **Lexical** mode skips the embedding call.
- Queries the storage: Dense uses the [HNSW](https://docs.tryllengine.com/v0.10/reference/glossary.html#hnsw) cosine index; Lexical uses an in-process BM25 index over record text (Path A load-or-builds a `.bm25` sidecar; Unicode casefold via utf8proc, format version 2); Hybrid runs both and fuses with Reciprocal Rank Fusion (`rrf_k`). The embedding model is owned by the KB config, not by this node.
- Applies `threshold` as a **maximum cosine distance on the dense leg** (unchanged in Dense mode). In Hybrid, threshold gates dense candidates *before* fusion — BM25-only hits are not gated. In Lexical, threshold is inert. Prefer `top_k` as the primary precision control in Hybrid.
- Attaches a knowledge block (`source` + surviving chunks) to the current [turn](https://docs.tryllengine.com/v0.10/reference/glossary.html#interaction). If nothing survives filtering, an empty block is still attached (the Mustache `{{#knowledge_<source>}}` section simply renders nothing). Lexical-only hits report `distance` as JSON `null` (no cosine distance).

The attached knowledge is rendered into the prompt **on the next Generate node**, via the `template` and `placement` params on that node.

## Diagnostics

When `enable_diagnostics = true`, this node contributes to `debug_info.nodes[].diagnostics`. See [Turn Diagnostics JSON](https://docs.tryllengine.com/v0.10/reference/turn-diagnostics-json.html) for the envelope.

No model runs here, so there is no `engine` section.

| Key                                  | Meaning                                                                                                                      | Absent when                                                                         |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| `parameters.source`                  | The `source` label attached to the component.                                                                                | —                                                                                   |
| `parameters.embedded_string_storage` | Configured storage name.                                                                                                     | Not set.                                                                            |
| `parameters.top_k`                   | Number of chunks requested.                                                                                                  | —                                                                                   |
| `parameters.threshold`               | Max cosine distance used for dense-leg filtering, or `"inf"` when disabled.                                                  | —                                                                                   |
| `parameters.retrieval_mode`          | `Dense` / `Lexical` / `Hybrid`.                                                                                              | —                                                                                   |
| `parameters.rrf_k`                   | RRF constant (Hybrid only).                                                                                                  | —                                                                                   |
| `parameters.filter`                  | Raw JSON of the active filter. Empty string when no filter is set.                                                           | —                                                                                   |
| `parameters.raw_result_count`        | Dense: chunks from the index before threshold. Hybrid: `dense_raw_count + sparse_raw_count`. Lexical: sparse hit count.      | —                                                                                   |
| `parameters.filtered_count`          | Dense/Hybrid: chunks removed by the dense-leg threshold. Lexical: always `0`.                                                | —                                                                                   |
| `parameters.dense_raw_count`         | Dense hits before threshold (0 for Lexical).                                                                                 | The run was pure Dense — the per-leg counts appear only on the Hybrid/Lexical path. |
| `parameters.dense_filtered_count`    | Dense hits removed by threshold.                                                                                             | As above.                                                                           |
| `parameters.sparse_raw_count`        | BM25 hits before fusion / top-K trim.                                                                                        | As above.                                                                           |
| `parameters.fused_candidate_count`   | Hybrid: unique record ids across both legs before top-K trim (0 for Lexical).                                                | As above.                                                                           |
| `parameters.result_count`            | Chunks actually attached to the component.                                                                                   | —                                                                                   |
| `input.query`                        | The text that was embedded / searched.                                                                                       | The resolved text was empty.                                                        |
| `output.results[]`                   | Ranked hits, in server order (RRF rank for Hybrid). **Always present** — an empty array on `not_found`, never a missing key. | —                                                                                   |
| `output.results[].id`                | Chunk id.                                                                                                                    | —                                                                                   |
| `output.results[].distance`          | Cosine distance (lower = more similar), or JSON `null` for lexical-only hits.                                                | —                                                                                   |
| `output.results[].bm25_score`        | Okapi BM25 score (stable public diagnostic).                                                                                 | The sparse leg did not contribute to this hit.                                      |
| `output.results[].rank_dense`        | 1-based dense-leg rank (stable public diagnostic).                                                                           | This hit did not come from the dense leg.                                           |
| `output.results[].rank_sparse`       | 1-based BM25-leg rank (stable public diagnostic).                                                                            | This hit did not come from the sparse leg.                                          |
| `output.results[].text`              | Chunk text that will be rendered into the prompt.                                                                            | —                                                                                   |

The count chain `raw_result_count` → `filtered_count` → `result_count` is the funnel to read when retrieval returns too little: it separates "the index had nothing" from "the threshold rejected it". Note that `output.results[]` lists only the surviving chunks — the near-misses are visible as the gap between the counts, and in the turn inspector's dimmed rows.

## Minimum working example

```
from tryll_client.graph import GraphDescription, GenerateParams, RetrieveParams, Placement

RAG_TEMPLATE = (
    "{{#knowledge}}"
    "{{name}}:\n{{#chunks}}- {{text}}\n{{/chunks}}\n"
    "{{/knowledge}}"
)

graph = (
    GraphDescription()
    .add_node("knowledge", RetrieveParams(
        embedded_string_storage="aquarium/aquarium_all_mini.json",
        top_k=3,
        threshold=0.6,
        found_exit="answer",
        not_found_exit="answer",
    ))
    .add_node("answer", GenerateParams(
        template=RAG_TEMPLATE,
        placement=Placement.BeforeUserAsSystem,
        default_exit="",   # empty = END
    ))
    .set_start_node("knowledge")
    .set_default_model_name("My Local Model")
)

agent = client.create_agent(graph)
```

```
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

RetrieveParamsT kp;
kp.embedded_string_storage = "aquarium/aquarium_all_mini.json";
kp.top_k                   = 3;
kp.threshold               = 0.6f;
kp.found_exit              = "answer";
kp.not_found_exit          = "answer";

GenerateParamsT gp;
gp.template_  = "{{#knowledge}}{{name}}:\n{{#chunks}}- {{text}}\n{{/chunks}}\n{{/knowledge}}";
gp.placement  = ::Tryll::Placement_BeforeUserAsSystem;
// gp.default_exit = ""; // empty = END (the default)

GraphDescription graph;
graph.AddRetrieve("knowledge", std::move(kp))
     .AddGenerate("answer",    std::move(gp))
     .SetStartNode("knowledge")
     .SetDefaultModelName("My Local Model");

auto agent = client.CreateAgent(graph);
```

```
using Tryll.Client;

var graph = new TryllGraphBuilder()
    .AddRetrieve("knowledge", new TryllRetrieveParams
    {
        EmbeddedStringStorage = "aquarium/aquarium_all_mini.json",
        TopK                  = 3,
        Threshold             = 0.6f,
        FoundExit             = "answer",
        NotFoundExit          = "answer",
    })
    .AddGenerate("answer", new TryllGenerateParams
    {
        Template  = "{{#knowledge}}{{name}}:\n{{#chunks}}- {{text}}\n{{/chunks}}\n{{/knowledge}}",
        Placement = TryllPlacement.BeforeUserAsSystem,
    })
    .SetStartNode("knowledge")
    .SetDefaultModelName("My Local Model")
    .Build();
```

```
#include "Generated/TryllGraphBuilder.Nodes.h"
#include "Generated/TryllNodeParamsFactory.h"

UTryllRetrieveParams* KP = UTryllNodeParamsFactory::MakeRetrieveParams(this);
KP->EmbeddedStringStorage.Path = TEXT("aquarium/aquarium_all_mini.json");
KP->TopK      = 3;
KP->Threshold = 0.6f;
KP->FoundExit    = TEXT("answer");
KP->NotFoundExit = TEXT("answer");

UTryllGenerateParams* GP = UTryllNodeParamsFactory::MakeGenerateParams(this);
GP->bOverrideTemplate = true;
GP->Template  = TEXT("{{#knowledge}}{{name}}:\n{{#chunks}}- {{text}}\n{{/chunks}}\n{{/knowledge}}");
GP->Placement = ETryllPlacement::BeforeUserAsSystem;

FTryllGraphDescription Graph = FTryllGraphBuilder()
    .AddNode(TEXT("knowledge"), KP)
    .AddNode(TEXT("answer"),    GP)
    .SetStartNode(TEXT("knowledge"))
    .SetDefaultModelName(TEXT("My Local Model"))
    .Build();
```

See the full walkthrough in [How to create a simple RAG assistant](https://docs.tryllengine.com/v0.10/how-to/create-simple-rag-assistant.html).

## Client bindings

- **C++:** `GraphDescription::AddRetrieve(name, RetrieveParamsT)` — `GraphDescription.h`
- **Python:** `GraphDescription.add_node(name, RetrieveParams(...))` — `tryll_client.graph`
- **Unity:** `TryllGraphBuilder.AddRetrieve(name, new TryllRetrieveParams{...})` — `Runtime/Generated/TryllGraphBuilder.Nodes.cs`
- **Unreal:** `AddRetrieveNode(builder, name, UTryllRetrieveParams*)` — `Generated/TryllGraphBuilder.Nodes.h`

## Related

- [Embedded String Storage](https://docs.tryllengine.com/v0.10/reference/embedded-string-storage.html)
- [Concept: RAG](https://docs.tryllengine.com/v0.10/concepts/rag.html)
- [How to create a simple RAG assistant](https://docs.tryllengine.com/v0.10/how-to/create-simple-rag-assistant.html)
- [How to use Mustache templates](https://docs.tryllengine.com/v0.10/how-to/use-mustache-templates.html)
- [Glossary](https://docs.tryllengine.com/v0.10/reference/glossary.html#retrieve-node)
