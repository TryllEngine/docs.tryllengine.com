# Create a Simple RAG Assistant

Prepare a small knowledge base, index it into an [embedded string storage](https://docs.tryllengine.com/reference/embedded-string-storage.html), build a graph with a `Retrieve` node in front of a `Generate` node, and get answers grounded in your data.

Prerequisites

- A connected session, created with the `LlamaCpp` engine.
- A **language model** available (e.g., `"My Local Model"`).
- An **embedding model** available (e.g., `"All-MiniLM-L6-v2 (Q4_K_M)"`). Add one to [`models.json`](https://docs.tryllengine.com/reference/model-management.html) with `model_type: "embedding"` if you do not have one yet.

## Step 1 — prepare the knowledge base

`CreateEmbeddedStringStorageRequest` has two construction paths, both documented in [Embedded String Storage → Creation paths](https://docs.tryllengine.com/reference/embedded-string-storage.html#creation-paths-explicit-pin):

- **Path A (file-backed)** — the client passes a `config_path`; the server reads a JSON config that points at a records file and an optional pre-built `.usearch` index. Best for **stable corpora**: the index is cached on disk and reused across runs.
- **Path B (inline strings)** — the client sends a `strings[]` array plus an `embedding_model` over the wire; the server embeds and builds the index in memory. Nothing is written to disk; the storage is gone when the session ends. Best for **small, ephemeral content** generated client-side.

See [Lifetime and Ownership](https://docs.tryllengine.com/concepts/lifetime-and-ownership.html#embeddedstringstorage) for how an embedded storage stays alive across agents and what happens when you destroy one mid-session.

This how-to uses **Path A**. Two files drive it:

**`my-docs.kb.json`** — the records:

```
[
  {
    "id": "rule-001",
    "text": "Players respawn at the nearest fast-travel beacon 10 seconds after death.",
    "metadata": {"category": "gameplay"}
  },
  {
    "id": "rule-002",
    "text": "Damage to allies is reduced by 80% but not zero, to prevent griefing while allowing friendly fire cues.",
    "metadata": {"category": "gameplay"}
  }
]
```

**`my-docs.json`** — the config pointing at it:

```
{
  "version": 1,
  "embedding_model": "All-MiniLM-L6-v2 (Q4_K_M)",
  "records_file": "my-docs.kb.json",
  "index_file":   "my-docs.kb.usearch"
}
```

Put both files alongside the server (say in `data/rag/`). The first creation will embed every record and write `my-docs.kb.usearch` to disk; subsequent runs skip the embedding pass.

How these paths resolve

Both the `config_path` you pass to `CreateEmbeddedStringStorage` and the `Retrieve` node's `embedded_string_storage` are resolved **relative to the session storage root** — `CreateSessionRequest.storage_data_folder` if set, else the server's `storage_root`, else the server's exe directory. Use the **same** relative path in both places (this how-to uses `data/rag/my-docs.json` throughout, for files sitting in `<exe>/data/rag/`).

## Step 2 — create the embedded string storage

```
var (info, error) = await TryllClient.Instance.RequestCreateEmbeddedStringStorageAsync(
    "my_docs", "data/rag/my-docs.json");
if (!error.IsOk) { Debug.LogError(error.Message); return; }
Debug.Log($"{info.RecordCount} records, {info.EmbeddingDim} dims");
```

Call `UTryllSubsystem::RequestCreateEmbeddedStringStorage` (C++) with `ConfigPath = "data/rag/my-docs.json"`. The completion callback receives `{Name, RecordCount, bSuccess}`. This entry point is not `BlueprintCallable` in the current plugin; call it from C++ game code.

```
auto info = client.CreateEmbeddedStringStorage(
    "my_docs",
    "data/rag/my-docs.json",
    /*embeddingModel=*/"",                  // use config's value
    std::chrono::minutes{10});
std::cout << info.recordCount << " records, "
          << info.embeddingDim << " dims\n";
```

```
info = client.create_embedded_string_storage(
    name="my_docs",
    config_path="data/rag/my-docs.json",
)
print(f"{info.record_count} records, {info.embedding_dim} dims")
```

First call builds the index; later calls with the same config reuse the on-disk `.usearch`. See [Embedded String Storage](https://docs.tryllengine.com/reference/embedded-string-storage.html) for the Path B (inline strings) variant.

## Step 3 — build the graph

```
flowchart LR
    retrieve["Retrieve<br>retrieve"]
    gen["Generate<br>answer"]
    refuse["CannedResponse<br>refuse"]
    retrieve -- "found"     --> gen
    retrieve -- "not_found" --> refuse
    gen    -- "default" --> END
    refuse -- "default" --> END
```

The `not_found` exit routes to a `CannedResponse` node that emits a pre-written "I don't know" line. This makes the empty case explicit in the graph — the model never runs when retrieval returns nothing, which is faster and gives the client a deterministic response.

First, create a small string storage to back the `CannedResponse` node (see [Use Canned Responses and Guardrails](https://docs.tryllengine.com/how-to/use-canned-responses-and-guardrails.html) for more on canned responses):

```
var error = await TryllClient.Instance.RequestCreateStringStorageAsync(
    "rag_not_found",
    new List<string> {
        "I don't have information on that in my knowledge base.",
        "I couldn't find anything about that in the docs I have.",
    });
if (!error.IsOk) Debug.LogError(error.Message);
```

```
client.CreateStringStorage("rag_not_found", {
    "I don't have information on that in my knowledge base.",
    "I couldn't find anything about that in the docs I have.",
});
```

```
client.create_string_storage(
    name="rag_not_found",
    strings=[
        "I don't have information on that in my knowledge base.",
        "I couldn't find anything about that in the docs I have.",
    ],
)
```

Then build the graph. The `Generate` node receives a Mustache template that renders the retrieved chunks as a system turn before the user's question:

```
const string ragTemplate =
    "{{#knowledge}}"
    + "{{name}}:\n{{#chunks}}- {{text}}\n{{/chunks}}"
    + "\n{{/knowledge}}";

var graph = new TryllGraphBuilder()
    .AddRetrieve("retrieve", new TryllRetrieveParams
    {
        EmbeddedStringStorage = "data/rag/my-docs.json",
        TopK                  = 3,
        Threshold             = 0.6f,
        FoundExit             = "answer",
        NotFoundExit          = "refuse",
    })
    .AddGenerate("answer", new TryllGenerateParams
    {
        Template     = ragTemplate,
        Placement    = TryllPlacement.BeforeUserAsSystem,
        SystemPrompt = "Answer using the context above. If the context does not contain the answer, say so.",
        // DefaultExit is "" (END) by default.
    })
    .AddCannedResponse("refuse", new TryllCannedResponseParams
    {
        StringStorage = "rag_not_found",
        // DefaultExit is "" (END) by default.
    })
    .SetStartNode("retrieve")
    .SetDefaultModelName("My Local Model")
    .Build();

var (agent, createError) = await TryllClient.Instance.RequestCreateAgentAsync(graph);
if (!createError.IsOk) Debug.LogError(createError.Message);

agent.SetOnAnswerText((nodeName, text, isDelta, isFinal) => { /* stream text */ });
agent.SetOnTurnComplete((status, debugInfo, tokens) => { /* turn ended */ });
```

```
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

constexpr std::string_view kRagTemplate =
    "{{#knowledge}}"
    "{{name}}:\n"
    "{{#chunks}}- {{text}}\n{{/chunks}}"
    "\n{{/knowledge}}";

RetrieveParamsT retrieveParams;
retrieveParams.embedded_string_storage = "data/rag/my-docs.json";
retrieveParams.top_k                   = 3;
retrieveParams.threshold               = 0.6f;
retrieveParams.found_exit              = "answer";
retrieveParams.not_found_exit          = "refuse";

GenerateParamsT answerParams;
answerParams.template_     = std::string{kRagTemplate};
answerParams.placement     = Placement::BeforeUserAsSystem;
answerParams.system_prompt = "Answer using the context above. If the context does not contain the answer, say so.";
// answerParams.default_exit is "" (END) by default.

CannedResponseParamsT refuseParams;
refuseParams.string_storage = "rag_not_found";
// refuseParams.default_exit is "" (END) by default.

GraphDescription graph;
graph.AddRetrieve("retrieve", std::move(retrieveParams))
     .AddGenerate("answer",   std::move(answerParams))
     .AddCannedResponse("refuse", std::move(refuseParams))
     .SetStartNode("retrieve")
     .SetDefaultModelName("My Local Model");

auto agent = client.CreateAgent(graph);
```

```
from tryll_client.graph import (
    GraphDescription, RetrieveParams, GenerateParams,
    CannedResponseParams, Placement,
)

RAG_TEMPLATE = (
    "{{#knowledge}}"
    "{{name}}:\n"
    "{{#chunks}}- {{text}}\n{{/chunks}}"
    "\n{{/knowledge}}"
)

graph = (
    GraphDescription()
    .add_node("retrieve", RetrieveParams(
        embedded_string_storage="data/rag/my-docs.json",
        top_k=3,
        threshold=0.6,
        found_exit="answer",
        not_found_exit="refuse",
    ))
    .add_node("answer", GenerateParams(
        template=RAG_TEMPLATE,
        placement=Placement.BeforeUserAsSystem,
        system_prompt="Answer using the context above. If the context does not contain the answer, say so.",
        # default_exit is "" (END) by default.
    ))
    .add_node("refuse", CannedResponseParams(
        string_storage="rag_not_found",
        # default_exit is "" (END) by default.
    ))
    .set_start_node("retrieve")
    .set_default_model_name("My Local Model")
)

agent = client.create_agent(graph)
```

## Step 4 — ask a grounded question

```
agentComp.SendMessage("What happens when players damage their teammates?");
```

Call `UTryllAgentComponent::SendMessage` with the prompt; bind **On Answer Text** to append streaming chunks to your UI widget and **On Turn Complete** to flip the "typing" indicator off.

```
agent.SetOnAnswerText([](std::string_view text, bool, bool)
    { std::cout << text << std::flush; });
agent.SendText("What happens when players damage their teammates?");
```

```
reply = agent.send_message(
    "What happens when players damage their teammates?"
)
print(reply)
```

You should get an answer that cites the "80% reduced friendly fire" rule from record `rule-002`.

## Verify it worked

Server log at `info` should show the `found` path:

```
[info] Node retrieve: found k=3 top_distance=0.18
[info] Node answer: default
```

For a query with no relevant records in the corpus, expect the `not_found` path instead — the `Generate` node is skipped and the client receives one of the `rag_not_found` lines:

```
[info] Node retrieve: not_found
[info] Node refuse: default
```

If you see `not_found` unexpectedly on a query you believe should hit the corpus, your `threshold` is too strict or the query is too unrelated to anything in `my-docs.kb.json`.

## Common pitfalls

- **Missing embedding model.** The KB config must declare `embedding_model`, and that model must already be downloaded — neither `CreateEmbeddedStringStorage` nor `CreateAgent` downloads embedding models for file-backed storages automatically. Preload it via the editor Model Manager or `DownloadModelRequest` before `CreateAgent`.
- **Stale index.** If you edited `my-docs.kb.json`, Tryll notices the newer mtime and rebuilds `.usearch`. Expect a one-time delay.
- **Answer ignores context.** Usually the `placement` is wrong for the model, or the template is not clear enough. See [Use Mustache Templates](https://docs.tryllengine.com/how-to/use-mustache-templates.html) for more options.
- **Top-K too small or threshold too tight.** Start with `top_k=3` and `threshold=0.6`, tune down while watching the retriever log.

## Next steps — filtering by metadata

If your knowledge base needs to gate records by player state (level, class, unlocked quests, …), add a `fields` array to your config and set the `filter` parameter on the `Retrieve` node. Update it at runtime with `change_params` whenever the relevant state changes.

### 1. Declare the metadata field in your KB config

```
{
  "version": 1,
  "embedding_model": "All-MiniLM-L6-v2 (Q4_K_M)",
  "records_file": "aquarium.kb.json",
  "index_file":   "aquarium.usearch",
  "fields": [
    { "name": "level", "type": "int", "default": 1 }
  ]
}
```

Each record in `aquarium.kb.json` carries the field in its `metadata` object:

```
[
  { "id": "lv1_clownfish", "text": "...", "metadata": { "level": 1 } },
  { "id": "lv3_lionfish",  "text": "...", "metadata": { "level": 3 } }
]
```

### 2. Set the filter when creating the agent

```
import json
from copy import deepcopy
from tryll_client.graph import GraphDescription, RetrieveParams, GenerateParams, Placement

retrieve_baseline = RetrieveParams(
    embedded_string_storage="aquarium/aquarium_all_mini.json",
    top_k=3,
    threshold=0.6,
    # gate by player level: level <= player_level
    filter=json.dumps({"op": "le",
                       "lhs": {"knowledge": "level"},
                       "rhs": {"value": player_level}}),
    found_exit="answer",
    not_found_exit="answer",
)

graph = (
    GraphDescription()
    .add_node("knowledge", retrieve_baseline)
    .add_node("answer", GenerateParams(
        template=RAG_TEMPLATE,
        placement=Placement.BeforeUserAsSystem,
        default_exit="",
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
// start at level 1: level <= 1
kp.filter                  = R"({"op":"le","lhs":{"knowledge":"level"},"rhs":{"value":1}})";
kp.found_exit              = "answer";
kp.not_found_exit          = "answer";
// ... build and create agent as before
```

### 3. Update the filter when state changes

```
from copy import deepcopy

def on_player_level_up(new_level: int):
    p = deepcopy(retrieve_baseline)
    p.filter = json.dumps({"op": "le",
                          "lhs": {"knowledge": "level"},
                          "rhs": {"value": new_level}})
    agent.change_params("knowledge", p)
```

```
void OnPlayerLevelUp(int newLevel, RetrieveParamsT& baseline, AgentProxy& agent)
{
    RetrieveParamsT p = baseline;
    p.filter = R"({"op":"le","lhs":{"knowledge":"level"},"rhs":{"value":)"
               + std::to_string(newLevel) + R"(}})";
    agent.ChangeParams("knowledge", std::move(p));
}
```

For the full filter expression grammar (AND/OR/NOT, set membership, numeric ranges) see [Retrieve filter grammar](https://docs.tryllengine.com/reference/nodes/retrieve-filter-grammar.html).

- [Reference: Embedded String Storage — Metadata schema](https://docs.tryllengine.com/reference/embedded-string-storage.html#metadata-schema)
- [Reference: Retrieve filter grammar](https://docs.tryllengine.com/reference/nodes/retrieve-filter-grammar.html)
- [How-to: Change Agent Parameters at Runtime](https://docs.tryllengine.com/how-to/change-agent-parameters.html)

## Related

- [Reference: Embedded String Storage](https://docs.tryllengine.com/reference/embedded-string-storage.html)
- [Reference: Retrieve node](https://docs.tryllengine.com/reference/nodes/retrieve.html)
- [Reference: Retrieve filter grammar](https://docs.tryllengine.com/reference/nodes/retrieve-filter-grammar.html)
- [Concept: RAG](https://docs.tryllengine.com/concepts/rag.html)
- [How-to: Use Mustache Templates](https://docs.tryllengine.com/how-to/use-mustache-templates.html)
- [How-to: Change Agent Parameters at Runtime](https://docs.tryllengine.com/how-to/change-agent-parameters.html)
