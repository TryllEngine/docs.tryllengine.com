# Build an Intent-Driven NPC

Use a small labelled knowledge base and a [`ClassifyIntent`](https://docs.tryllengine.com/reference/nodes/classify-intent.html) → [`IntentToInstruction`](https://docs.tryllengine.com/reference/nodes/intent-to-instruction.html) → [`Generate`](https://docs.tryllengine.com/reference/nodes/generate.html) pipeline to give an NPC distinct, tone-appropriate replies for recognised player intents (insult, compliment, joke, …) while falling back gracefully to a general response for anything unclassified.

Prerequisites

- A connected session, created with the `LlamaCpp` engine.
- A **language model** and an **embedding model** available (e.g. `"Llama 3.2 3B Instruct (Q4_K_M)"` and `"All-MiniLM-L6-v2 (Q4_K_M)"`). Add an embedding model to [`models.json`](https://docs.tryllengine.com/reference/model-management.html) with `model_type: "embedding"` if you do not have one yet.

______________________________________________________________________

## Overview

```
flowchart LR
    classify["ClassifyIntent\nclassify"]
    pick["IntentToInstruction\npick"]
    compose["Transform\ncompose"]
    respond["Generate\nrespond"]
    classify -- "found" --> pick
    classify -- "not_found" --> compose
    pick -- "default" --> compose
    compose -- "default" --> respond
    respond -- "default" --> END
```

The `ClassifyIntent` node embeds the user's message, searches a small KB of labelled example phrases (5–10 per intent is enough), and attaches an `IntentionComponent` on `"found"`. `IntentToInstruction` looks up that label in a `Map` string storage to retrieve a tone directive, attaches it as an instruction [slot](https://docs.tryllengine.com/concepts/slots-and-inter-node-value-passing.html). A `Transform` node then folds that directive above the user message via Mustache into its own slot, and `Generate` consumes it via `input` — this is the pattern for fully replacing what the model sees (the old `in_place_of_user` placement). If the intent is unknown (`"not_found"`), `Transform` still runs (with an empty `{{#instructions}}` block), so `Generate` always reads from the same `compose` slot — the NPC answers naturally without any override.

______________________________________________________________________

## Step 1 — prepare the intent knowledge base

Create two files alongside the server.

**`guard_intents.kb.json`** — labelled example phrases (5 per intent):

```
[
  { "id": "insult_1", "text": "You are an absolute idiot, move aside.",  "metadata": { "intent": "insult" } },
  { "id": "insult_2", "text": "Out of my way, you worthless fool.",       "metadata": { "intent": "insult" } },
  { "id": "insult_3", "text": "What a pathetic excuse for a guard.",      "metadata": { "intent": "insult" } },
  { "id": "insult_4", "text": "You are dumber than a bag of rocks.",      "metadata": { "intent": "insult" } },
  { "id": "insult_5", "text": "Move, you incompetent buffoon.",            "metadata": { "intent": "insult" } },

  { "id": "pleasing_1", "text": "Good evening, kind sir, may I please pass?",     "metadata": { "intent": "pleasing" } },
  { "id": "pleasing_2", "text": "I humbly request entry into the city, good guard.", "metadata": { "intent": "pleasing" } },
  { "id": "pleasing_3", "text": "You look like a fine guardian. Might you let me through?", "metadata": { "intent": "pleasing" } },
  { "id": "pleasing_4", "text": "Thank you for your service. Could I trouble you for passage?", "metadata": { "intent": "pleasing" } },
  { "id": "pleasing_5", "text": "I appreciate your diligence. Would it be possible to enter?", "metadata": { "intent": "pleasing" } },

  { "id": "joking_1", "text": "Why did the orc cross the road? To get to the other fortress!", "metadata": { "intent": "joking" } },
  { "id": "joking_2", "text": "A paladin, a rogue, and a wizard walk into a tavern… ouch.",  "metadata": { "intent": "joking" } },
  { "id": "joking_3", "text": "How many guards does it take to open a gate?",                 "metadata": { "intent": "joking" } },
  { "id": "joking_4", "text": "Knock knock. Who's there? A very impatient traveller.",        "metadata": { "intent": "joking" } },
  { "id": "joking_5", "text": "I've got a joke about a magic door but I'm not sure it will open.", "metadata": { "intent": "joking" } }
]
```

**`guard_intents.json`** — the config pointing at it:

```
{
  "version": 1,
  "embedding_model": "All-MiniLM-L6-v2 (Q4_K_M)",
  "records_file":    "guard_intents.kb.json",
  "index_file":      "guard_intents.usearch",
  "fields": [
    { "name": "intent", "type": "string", "default": "" }
  ]
}
```

The `fields` array is required — `ClassifyIntent` validates that the field referenced by `metadata_field` exists and is `string`-typed. On first run the server embeds every record and writes `guard_intents.usearch` to disk; subsequent runs skip the embedding pass.

______________________________________________________________________

## Step 2 — create the embedded string storage

```
info = client.create_embedded_string_storage(
    name="guard_intent_kb",
    config_path="data/guard_intents.json",
)
print(f"{info.record_count} records, {info.embedding_dim} dims")
```

```
auto info = client.CreateEmbeddedStringStorage(
    "guard_intent_kb",
    "data/guard_intents.json");
```

______________________________________________________________________

## Step 3 — create the instruction map

The instruction map is a `Map`-kind string storage: keys are intent labels, values are tone directives fed to the model as invisible instructions.

```
client.create_keyed_string_storage(
    name="guard_instructions",
    keys=  ["insult",  "pleasing",  "joking"],
    values=[
        "The traveller has insulted you. Respond curtly and insult them back — short and sharp.",
        "The traveller is being polite. Stay grumpy but show grudging satisfaction. One sentence.",
        'The traveller told a joke. Permit yourself one brief laugh (e.g. "Ha.") then immediately refocus on duty.',
    ],
)
```

```
client.CreateStringStorageKeyed(
    "guard_instructions",
    /*keys=*/   {"insult", "pleasing", "joking"},
    /*values=*/ {
        "The traveller has insulted you. Respond curtly and insult them back — short and sharp.",
        "The traveller is being polite. Stay grumpy but show grudging satisfaction. One sentence.",
        "The traveller told a joke. Permit yourself one brief laugh then refocus on duty.",
    },
    ::Tryll::StringStorageKind_Map);
```

```
await TryllClient.Instance.RequestCreateKeyedStringStorageAsync(
    "guard_instructions",
    new List<string> { "insult", "pleasing", "joking" },
    new List<string> {
        "The traveller has insulted you. Respond curtly and insult them back — short and sharp.",
        "The traveller is being polite. Stay grumpy but show grudging satisfaction. One sentence.",
        "The traveller told a joke. Permit yourself one brief laugh then refocus on duty.",
    },
    Tryll.StringStorageKind.Map);
```

```
auto* Subsystem = GetGameInstance()->GetSubsystem<UTryllSubsystem>();
Subsystem->RequestCreateStringStorageKeyed(
    TEXT("guard_instructions"),
    /*Keys=*/   { TEXT("insult"), TEXT("pleasing"), TEXT("joking") },
    /*Values=*/ {
        TEXT("The traveller has insulted you. Respond curtly and insult them back — short and sharp."),
        TEXT("The traveller is being polite. Stay grumpy but show grudging satisfaction. One sentence."),
        TEXT("The traveller told a joke. Permit yourself one brief laugh then refocus on duty."),
    },
    /*Kind=*/ 1); // 1 = Map
```

Or reference a Map file by path

Instead of creating the storage over the wire, you can drop a `.json` Map file (an array of `{ "id": "<key>", "text": "<value>" }` records) under your storage root and point `IntentToInstruction.string_storage` at its relative path — the server loads it at agent-creation time. `.json` → Map, `.mm.json` → Multimap, `.txt` → List. See the [String Storage reference](https://docs.tryllengine.com/reference/string-storage.html).

______________________________________________________________________

## Step 4 — build the graph

The `Transform` node (`compose`) renders the tone directive (if found) as a hidden system-style line above the user message into its own slot, and the `Generate` node (`respond`) consumes that slot via `input="compose"`. When no intent was matched the `{{#instructions}}` block is empty and the model answers freely.

```
from tryll_client.graph import (
    GraphDescription, ClassifyIntentParams, IntentToInstructionParams,
    TransformParams, GenerateParams,
)

SYSTEM_PROMPT = (
    "You are roleplaying as Aldric, a grumpy gate guard. "
    "Stay in character at all times. Use short sentences."
)

TEMPLATE = (
    "{{#instructions}}[Tone directive: {{text}}]\n{{/instructions}}"
    "Traveller says: {{user_message}}"
)

graph = (
    GraphDescription()
    .add_node("classify", ClassifyIntentParams(
        embedded_string_storage="guard_intents/guard_intents.json",
        metadata_field="intent",
        threshold=0.6,
        found_exit="pick",
        not_found_exit="compose",
    ))
    .add_node("pick", IntentToInstructionParams(
        string_storage="guard_instructions",
        output_name="tone",
        default_exit="compose",
    ))
    .add_node("compose", TransformParams(
        template=TEMPLATE,
        default_exit="respond",
    ))
    .add_node("respond", GenerateParams(
        system_prompt=SYSTEM_PROMPT,
        input="compose",
        # default_exit is "" (END) by default.
    ))
    .set_start_node("classify")
    .set_default_model_name("Llama 3.2 3B Instruct (Q4_K_M)")
)

agent = client.create_agent(graph)
```

```
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

constexpr const char* kSystemPrompt =
    "You are roleplaying as Aldric, a grumpy gate guard. "
    "Stay in character at all times. Use short sentences.";

constexpr const char* kTemplate =
    "{{#instructions}}[Tone directive: {{text}}]\n{{/instructions}}"
    "Traveller says: {{user_message}}";

ClassifyIntentParamsT classifyParams;
classifyParams.embedded_string_storage = "guard_intents/guard_intents.json";
classifyParams.metadata_field          = "intent";
classifyParams.threshold               = 0.6f;
classifyParams.found_exit              = "pick";
classifyParams.not_found_exit          = "compose";

IntentToInstructionParamsT pickParams;
pickParams.string_storage = "guard_instructions";
pickParams.output_name    = "tone";
pickParams.default_exit   = "compose";

TransformParamsT composeParams;
composeParams.template_    = kTemplate;
composeParams.default_exit = "respond";

GenerateParamsT respondParams;
respondParams.system_prompt = kSystemPrompt;
respondParams.input         = "compose";
// respondParams.default_exit is "" (END) by default.

GraphDescription graph;
graph.AddClassifyIntent("classify", std::move(classifyParams))
     .AddIntentToInstruction("pick",    std::move(pickParams))
     .AddTransform("compose",          std::move(composeParams))
     .AddGenerate("respond",            std::move(respondParams))
     .SetStartNode("classify")
     .SetDefaultModelName("Llama 3.2 3B Instruct (Q4_K_M)");

auto agent = client.CreateAgent(graph);
```

```
const string systemPrompt =
    "You are roleplaying as Aldric, a grumpy gate guard. " +
    "Stay in character at all times. Use short sentences.";

const string template =
    "{{#instructions}}[Tone directive: {{text}}]\n{{/instructions}}" +
    "Traveller says: {{user_message}}";

var graph = new TryllGraphBuilder()
    .AddClassifyIntent("classify", new TryllClassifyIntentParams
    {
        EmbeddedStringStorage = "guard_intents/guard_intents.json",
        MetadataField         = "intent",
        Threshold             = 0.6f,
        FoundExit             = "pick",
        NotFoundExit          = "compose",
    })
    .AddIntentToInstruction("pick", new TryllIntentToInstructionParams
    {
        StringStorage = "guard_instructions",
        OutputName    = "tone",
        DefaultExit   = "compose",
    })
    .AddTransform("compose", new TryllTransformParams
    {
        Template    = template,
        DefaultExit = "respond",
    })
    .AddGenerate("respond", new TryllGenerateParams
    {
        SystemPrompt = systemPrompt,
        Input        = "compose",
        // DefaultExit is "" (END) by default.
    })
    .SetStartNode("classify")
    .SetDefaultModelName("Llama 3.2 3B Instruct (Q4_K_M)")
    .Build();

var (agent, error) = await TryllClient.Instance.RequestCreateAgentAsync(graph);
```

```
#include "Generated/TryllGraphBuilder.Nodes.h"
#include "Generated/TryllNodeParamsFactory.h"

UTryllClassifyIntentParams* ClassifyP = UTryllNodeParamsFactory::MakeClassifyIntentParams(this);
ClassifyP->EmbeddedStringStorage = TEXT("guard_intents/guard_intents.json");
ClassifyP->MetadataField         = TEXT("intent");
ClassifyP->Threshold             = 0.6f;
ClassifyP->FoundExit             = TEXT("pick");
ClassifyP->NotFoundExit          = TEXT("compose");

UTryllIntentToInstructionParams* PickP = UTryllNodeParamsFactory::MakeIntentToInstructionParams(this);
PickP->StringStorage = TEXT("guard_instructions");
PickP->OutputName    = TEXT("tone");
PickP->DefaultExit   = TEXT("compose");

UTryllTransformParams* ComposeP = UTryllNodeParamsFactory::MakeTransformParams(this);
ComposeP->bOverrideTemplate = true;
ComposeP->Template = TEXT("{{#instructions}}[Tone directive: {{text}}]\n{{/instructions}}Traveller says: {{user_message}}");
ComposeP->DefaultExit = TEXT("respond");

UTryllGenerateParams* RespondP = UTryllNodeParamsFactory::MakeGenerateParams(this);
RespondP->bOverrideSystemPrompt = true;
RespondP->SystemPrompt = TEXT("You are roleplaying as Aldric, a grumpy gate guard. Stay in character at all times. Use short sentences.");
RespondP->bOverrideInput = true;
RespondP->Input = TEXT("compose");
// RespondP->DefaultExit is TEXT("") (END) by default.

FTryllGraphDescription Graph = FTryllGraphBuilder()
    .AddNode(TEXT("classify"), ClassifyP)
    .AddNode(TEXT("pick"),     PickP)
    .AddNode(TEXT("compose"),  ComposeP)
    .AddNode(TEXT("respond"),  RespondP)
    .SetStartNode(TEXT("classify"))
    .SetDefaultModelName(TEXT("Llama 3.2 3B Instruct (Q4_K_M)"))
    .Build();
```

______________________________________________________________________

## Step 5 — send messages and observe intents

```
# insult → found → pick → respond (with insult directive)
print(agent.send_message("Move aside, you dull-witted oaf."))

# pleasing → found → pick → respond (with polite directive)
print(agent.send_message("Good day to you, honourable guard."))

# joking → found → pick → respond (with joke directive)
print(agent.send_message("Why did the troll apply to be a guard? He heard it was rocky!"))

# neutral → not_found → respond (general answer, no directive)
print(agent.send_message("How far is it to the next town?"))
```

______________________________________________________________________

## Verify it worked

With `enable_diagnostics = True` on the agent, each `TurnComplete.debug_info` JSON will include:

```
{
  "classify.result.intent":        "insult",
  "classify.raw_distance":         "0.21",
  "classify.threshold":            "0.6",
  "pick.intent":                   "insult",
  "pick.found":                    "true",
  "pick.result_text":              "The traveller has insulted you. Respond curtly..."
}
```

For a neutral message (`not_found` path):

```
{
  "classify.result.intent": "",
  "classify.raw_distance":  ""
}
```

Server logs at `info` show:

```
[info] Node classify: found intent=insult distance=0.21
[info] Node pick: default found=true
[info] Node respond: default
```

______________________________________________________________________

## Common pitfalls

- **Threshold too tight.** Start with `0.6`. If clearly-labelled phrases fall through to `not_found`, widen to `0.7` and add more diverse examples to the KB.
- **Too few examples per intent.** Five phrases per intent is a minimum. More variation (formal/informal, long/short) improves classification.
- **Missing embedding model in KB config.** The `*.json` config must declare `embedding_model`. Preload that model before `CreateAgent` — graph creation does not auto-download embedding models for file-backed storages.
- **Wrong metadata field type.** `ClassifyIntent` rejects fields that are not `string`-typed at agent-creation time. Check your `fields` array in the config.
- **`not_found` path not wired.** Both `found` and `not_found` must be wired — an unwired exit is a graph compilation error.
- **Instruction never rendered.** Ensure the `Transform` node's template contains `{{#instructions}}…{{/instructions}}` or `{{slot.tone}}`. Without it, the instruction is attached but never inserted into the prompt.

______________________________________________________________________

## Next steps

- **Add more intents** — extend the KB with new example phrases and add a matching key/value pair to the instruction map.
- **Tune threshold at runtime** — use [`ChangeParam`](https://docs.tryllengine.com/how-to/change-agent-parameters.html) to adjust `classify.threshold` while the agent is running.
- **Filter by metadata** — combine `metadata_field` classification with a `filter` param to restrict which records compete during search (e.g. filter to a specific NPC or scene).

______________________________________________________________________

## Related

- [Reference: Classify Intent node](https://docs.tryllengine.com/reference/nodes/classify-intent.html)
- [Reference: Intent to Instruction node](https://docs.tryllengine.com/reference/nodes/intent-to-instruction.html)
- [Reference: String Storage](https://docs.tryllengine.com/reference/string-storage.html)
- [Reference: Embedded String Storage](https://docs.tryllengine.com/reference/embedded-string-storage.html)
- [How-to: Use Mustache Templates](https://docs.tryllengine.com/how-to/use-mustache-templates.html)
- [How-to: Change Agent Parameters at Runtime](https://docs.tryllengine.com/how-to/change-agent-parameters.html)
- [Concept: Workflows, graphs, and nodes](https://docs.tryllengine.com/concepts/workflows-graphs-nodes.html)
