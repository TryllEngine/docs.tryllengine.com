# Give an NPC a Memory It Can Search

Let a character store what the player tells it and find it again on a later turn, even after the game is saved and loaded. The graph searches the character's memory with a [`Retrieve`](https://docs.tryllengine.com/v0.10/reference/nodes/retrieve.html) node, replies, and then stores the player's line with a [`WriteToMemory`](https://docs.tryllengine.com/v0.10/reference/nodes/write-to-memory.html) node. All four clients can build this graph and save the store. The editor **Knowledge** window is Unity and Unreal only.

Experimental

Writable memory is experimental (protocol v20). The node parameters, the store API, and the TMKB save format may change between releases.

Prerequisites

- A connected session with `CreateSession` already called. See [Connect and Manage a Session](https://docs.tryllengine.com/v0.10/how-to/connect-and-manage-session.html).
- A language model and the embedding model on disk, for example `"Llama 3.2 3B Instruct (Q4_K_M)"` and `"All-MiniLM-L6-v2 (Q4_K_M)"`. Models are never downloaded for you. Acquire both first, with the [Model Manager](https://docs.tryllengine.com/v0.10/how-to/manage-models-in-editor.html) or as described in [Use Your Own Local Model](https://docs.tryllengine.com/v0.10/how-to/use-your-own-local-model.html). A missing model fails agent creation.

## The pattern

```
flowchart LR
    memory["Retrieve<br>memory<br>(From Name)"]
    reply["Generate<br>reply"]
    remember["WriteToMemory<br>remember"]
    memory -- "found / not_found" --> reply
    reply -- "default" --> remember
    remember -- "written / failed" --> END
```

`memory` searches the store for lines that match the player's message. `reply` sees those lines in its prompt and answers. `remember` then stores the player's message, so a later turn can find it. Both memory nodes name the same store and the same schema.

## Steps

### 1. Pick a bundled schema

A schema sets the store's embedding model and how many records it can hold. The server reads the catalog from `data/schemas.json`. It ships three schemas:

| Schema         | Embedding model             | `max_records` |
| -------------- | --------------------------- | ------------- |
| `npc_memory`   | `All-MiniLM-L6-v2 (Q4_K_M)` | 256           |
| `player_log`   | `All-MiniLM-L6-v2 (Q4_K_M)` | 512           |
| `shared_notes` | `All-MiniLM-L6-v2 (Q4_K_M)` | 256           |

The three differ only in their record cap. Pick the name that says what the store is for. This guide uses `npc_memory` for one character's memory.

Pick a store name too. One store per character works well, for example `npc.memory.nessa`. A name may use letters, digits, `_`, `.` and `-`, up to 128 characters. `storage_name` may include agent variables (`var.*`), which resolve once, when the agent is created.

In the Unity Inspector, **Schema** is a dropdown filled from the bundled catalog. An unknown value shows as **Unregistered**. The server has the final say either way.

### 2. Retrieve before the reply

Add a `Retrieve` node as the start node. Set `source_kind` to `FromName`, then set `storage_name` and `schema`. Leave `embedded_string_storage` empty: it is for prebuilt knowledge files only, and setting it with `FromName` fails agent creation.

Route both `found` and `not_found` to the reply. On the first turn the store is empty, so `not_found` is the normal path.

The reply reads the hits through its Mustache `template`. With `source` left empty, the section is named after the node, so a node named `memory` fills `{{#knowledge_memory}}`. See [Use Mustache Templates](https://docs.tryllengine.com/v0.10/how-to/use-mustache-templates.html).

### 3. Write to memory in the same turn

Add a `WriteToMemory` node after the reply, on the **same** `storage_name` and `schema`. Two nodes that name one store with different schemas fail agent creation.

- **`input`.** Leave it blank to store the player's message. Set it to an earlier node's output name to store that text instead, for example a hidden `Generate` that rewrites the line as a standalone fact. The node writes nothing only when the resolved text is empty; it still takes `written`.
- **`id`.** Leave it blank to add a new record on every write. Set it to store one record under a fixed id, such as `player_name`; writing that id again replaces the record. `id` may include agent variables, which resolve on each turn.
- **`failed`.** The node takes `failed` when the store rejects the write (store full or schema mismatch) or the `id` template fails. Leave it empty to end the turn, or route it to a node that logs or reacts.

The agent creates the store when it is created, if the store does not exist yet. Problems with the store are never an exit: an empty `schema`, an unknown schema, a bad store name, or a store that already exists under another schema fails agent creation, and the error message names the node.

```
using Tryll.Client;

const string Store  = "npc.memory.nessa";
const string Schema = "npc_memory";

var graph = new TryllGraphBuilder()
    .AddRetrieve("memory", new TryllRetrieveParams
    {
        SourceKind   = TryllKnowledgeSourceKind.FromName,
        StorageName  = Store,
        Schema       = Schema,
        TopK         = 3,
        Threshold    = 0.7f,
        FoundExit    = "reply",
        NotFoundExit = "reply",
    })
    .AddGenerate("reply", new TryllGenerateParams
    {
        SystemPrompt = "You are Nessa, the innkeeper. Use what you remember about the player.",
        Template     = "What you remember:\n{{#knowledge_memory}}- {{text}}\n{{/knowledge_memory}}",
        DefaultExit  = "remember",
    })
    .AddWriteToMemory("remember", new TryllWriteToMemoryParams
    {
        StorageName = Store,
        Schema      = Schema,
        // Input left empty: store the player's message.
        // WrittenExit and FailedExit left empty: end the turn.
    })
    .SetStartNode("memory")
    .SetDefaultModelName("Llama 3.2 3B Instruct (Q4_K_M)")
    .Build();
```

```
#include "Generated/TryllGraphBuilder.Nodes.h"
#include "Generated/TryllNodeParamsFactory.h"

const FString Store  = TEXT("npc.memory.nessa");
const FString Schema = TEXT("npc_memory");

UTryllRetrieveParams* MemoryP = UTryllNodeParamsFactory::MakeRetrieveParams(this);
MemoryP->SourceKind   = ETryllKnowledgeSourceKind::FromName;
MemoryP->StorageName  = Store;
MemoryP->Schema       = Schema;
MemoryP->TopK         = 3;
MemoryP->Threshold    = 0.7f;
MemoryP->FoundExit    = TEXT("reply");
MemoryP->NotFoundExit = TEXT("reply");

UTryllGenerateParams* ReplyP = UTryllNodeParamsFactory::MakeGenerateParams(this);
ReplyP->bOverrideSystemPrompt = true;
ReplyP->SystemPrompt = TEXT("You are Nessa, the innkeeper. Use what you remember about the player.");
ReplyP->bOverrideTemplate = true;
ReplyP->Template     = TEXT("What you remember:\n{{#knowledge_memory}}- {{text}}\n{{/knowledge_memory}}");
ReplyP->DefaultExit  = TEXT("remember");

UTryllWriteToMemoryParams* RememberP = UTryllNodeParamsFactory::MakeWriteToMemoryParams(this);
RememberP->StorageName = Store;
RememberP->Schema      = Schema;
// Input left empty: store the player's message.

FTryllGraphBuilder Builder;
AddRetrieveNode(Builder, TEXT("memory"), MemoryP);
AddGenerateNode(Builder, TEXT("reply"), ReplyP);
AddWriteToMemoryNode(Builder, TEXT("remember"), RememberP);
FTryllGraphDescription Graph = Builder
    .SetStartNode(TEXT("memory"))
    .SetDefaultModelName(TEXT("Llama 3.2 3B Instruct (Q4_K_M)"))
    .Build();
```

```
#include <tryll/TryllClient.h>

using namespace Tryll::Client;
using namespace Tryll::NodeParams;

const std::string store  = "npc.memory.nessa";
const std::string schema = "npc_memory";

RetrieveParamsT memoryP;
memoryP.source_kind    = ::Tryll::KnowledgeSourceKind_FromName;
memoryP.storage_name   = store;
memoryP.schema         = schema;
memoryP.top_k          = 3;
memoryP.threshold      = 0.7f;
memoryP.found_exit     = "reply";
memoryP.not_found_exit = "reply";

GenerateParamsT replyP;
replyP.system_prompt = "You are Nessa, the innkeeper. Use what you remember about the player.";
replyP.template_     = "What you remember:\n{{#knowledge_memory}}- {{text}}\n{{/knowledge_memory}}";
replyP.default_exit  = "remember";

WriteToMemoryParamsT rememberP;
rememberP.storage_name = store;
rememberP.schema       = schema;
// input left empty: store the player's message.

GraphDescription graph;
graph.AddRetrieve("memory", std::move(memoryP))
     .AddGenerate("reply", std::move(replyP))
     .AddWriteToMemory("remember", std::move(rememberP))
     .SetStartNode("memory")
     .SetDefaultModelName("Llama 3.2 3B Instruct (Q4_K_M)");

auto agent = client.CreateAgent(graph);
```

```
from tryll_client.graph import (
    GraphDescription, GenerateParams, KnowledgeSourceKind,
    RetrieveParams, WriteToMemoryParams,
)

STORE, SCHEMA = "npc.memory.nessa", "npc_memory"

graph = (
    GraphDescription()
    .add_node("memory", RetrieveParams(
        source_kind=KnowledgeSourceKind.FromName,
        storage_name=STORE, schema=SCHEMA,
        top_k=3, threshold=0.7,
        found_exit="reply", not_found_exit="reply",
    ))
    .add_node("reply", GenerateParams(
        system_prompt="You are Nessa, the innkeeper. Use what you remember about the player.",
        template="What you remember:\n{{#knowledge_memory}}- {{text}}\n{{/knowledge_memory}}",
        default_exit="remember",
    ))
    .add_node("remember", WriteToMemoryParams(
        storage_name=STORE, schema=SCHEMA,
        # input left empty: store the player's message.
    ))
    .set_start_node("memory")
    .set_default_model_name("Llama 3.2 3B Instruct (Q4_K_M)")
)

agent = client.create_agent(graph)
```

### 4. Read it back on a later turn

A write never refreshes knowledge that was already put into the prompt that turn. `memory` runs before `remember`, so the line the player says now is found from the next turn on.

1. Send a line with a fact, for example *"My name is Arin and I am scared of spiders."* `memory` finds nothing, and `remember` stores the line.
1. Send a question about it, for example *"What am I afraid of?"* `memory` finds the stored line, and `reply` sees it under *What you remember*.

The chat history can also hold the answer while the conversation is short. To prove the answer comes from memory, create a second agent with the same graph in the same session. It has no history, but it opens the same store, because a store lasts for the session. Ask it the question.

If the line is not found, lower the bar: raise `threshold` (a larger cosine distance counts as a match) or raise `top_k`. Check what the store returns for a phrase in the editor (step 6) before you change the prompt.

### 5. Save and load the store

The store lives only as long as the session. To keep it across a game save, write every store in the session to one TMKB file with `SaveGameMemory`, and read it back with `LoadGameMemory`.

- Every agent must be idle for both calls: no turn running, not paused, no KV-cache operation. Otherwise the call fails with [`8101 StorageBusy`](https://docs.tryllengine.com/v0.10/reference/error-codes.html#8xxx-embedded-string-storage).
- A relative path joins the server's save root. See [Server Configuration → Saves](https://docs.tryllengine.com/v0.10/reference/server-configuration.html#saves).
- `LoadGameMemory` needs an embed timing. `Lazy` loads the text now and builds the vectors on the first search. `Now` builds every vector during the load.
- Load replaces the records of each store in the file. A live store that is not in the file stays, and the result carries a warning.
- On load, recreate the agents first, then call `LoadGameMemory`. The agents see the loaded records on their next turn.

Save the dialog too, with the same save id, so a mismatched pair warns on load. See [Save and Load Agent State](https://docs.tryllengine.com/v0.10/how-to/save-and-load-agent-state.html).

```
var client = TryllClient.Instance;

var (saved, saveError) = await client.SaveGameMemoryAsync("memory/slot-1.tmkb", "slot-1");
if (!saveError.IsOk) Debug.LogError(saveError.Message);

// Later, in a new session, after the agents are recreated:
var (loaded, loadError) = await client.LoadGameMemoryAsync(
    "memory/slot-1.tmkb", TryllMutableKbEmbedTiming.Lazy);
if (!loadError.IsOk) Debug.LogError(loadError.Message);
else foreach (var warning in loaded.Warnings) Debug.LogWarning(warning);
```

```
UTryllSubsystem* Subsystem = GetGameInstance()->GetSubsystem<UTryllSubsystem>();

Subsystem->RequestSaveGameMemory(TEXT("memory/slot-1.tmkb"), TEXT("slot-1"),
    [](FTryllGameMemorySaveResult Result, FTryllError Error)
    {
        if (!Error.IsOk()) UE_LOG(LogTemp, Error, TEXT("%s"), *Error.Message);
    });

// Later, in a new session, after the agents are recreated:
Subsystem->RequestLoadGameMemory(TEXT("memory/slot-1.tmkb"),
    ETryllMutableKbEmbedTiming::Lazy, ETryllMutableKbCompatibility::Strict,
    [](FTryllGameMemoryLoadResult Result, FTryllError Error)
    {
        if (!Error.IsOk()) UE_LOG(LogTemp, Error, TEXT("%s"), *Error.Message);
        for (const FString& Warning : Result.Warnings)
            UE_LOG(LogTemp, Warning, TEXT("%s"), *Warning);
    });
```

```
client.SaveGameMemory("memory/slot-1.tmkb", "slot-1");

// Later, in a new session, after the agents are recreated:
auto result = client.LoadGameMemory(
    "memory/slot-1.tmkb", MutableKbEmbedTiming::Lazy);
for (const auto& warning : result.warnings)
    std::cerr << warning << "\n";
```

```
from tryll_client.mkb_snapshot import MutableKbEmbedTiming

client.save_game_memory("memory/slot-1.tmkb", "slot-1")

# Later, in a new session, after the agents are recreated:
result = client.load_game_memory("memory/slot-1.tmkb", MutableKbEmbedTiming.Lazy)
for warning in result.warnings:
    print(warning)
```

### 6. Check it in the editor

Run the workflow in the editor [Chat window](https://docs.tryllengine.com/v0.10/how-to/test-agent-in-editor.html), then open the **Knowledge** window: **Window → Tryll → Knowledge** in Unity, or **Window → Tryll → Tryll Knowledge** in Unreal. It shows the stores of the shared editor session, the one Chat uses. A game build or a Python client has its own session, so its stores do not appear there.

1. Click **Refresh**. The window never polls.
1. Select `npc.memory.nessa`. The summary line shows the schema, the live record count, and the write generation. A higher `gen` means something wrote to the store.
1. Switch to **Query**, type a phrase the player might say, and click **Run query**. The hits are the same kind of search `Retrieve` runs, best first, with their distance.
1. Use **Save game memory** and **Load game memory** to test the same file your game writes.

See [Inspect and Edit Knowledge Stores](https://docs.tryllengine.com/v0.10/how-to/inspect-knowledge-stores.html) for every field in the window.

From C++ or Python, query the store directly instead:

```
auto store = client.EnsureMutableKb("npc.memory.nessa", "npc_memory");
for (const auto& hit : store.Query("What am I afraid of?", 3))
    std::cout << hit.distance << "  " << hit.text << "\n";
```

```
store = client.ensure_mutable_kb("npc.memory.nessa", "npc_memory")
for hit in store.query("What am I afraid of?", top_k=3):
    print(f"{hit.distance:.3f}  {hit.text}")
```

## Limits of this experimental store

- **It is capped.** Each schema has a `max_records` cap. When the store is full, a new record is rejected and `WriteToMemory` takes `failed`. Nothing is evicted for you. Replacing an existing `id` still works.
- **It stores text only.** A record is an id and a line of text. The node writes no metadata.
- **The embedding model is fixed by the schema.** All three bundled schemas use `All-MiniLM-L6-v2 (Q4_K_M)`. You cannot pick another model per node.
- **It lasts for the session.** Disconnecting drops every store unless you saved it. Stores are per session, so a game build and the editor never share one.
- **A save holds text, not vectors.** Loading re-embeds every record, either during the load (`Now`) or on the first search (`Lazy`). Budget for that time on a loading screen.
- **New lines appear on the next turn.** A write does not refresh knowledge already projected in the current turn.

## Common pitfalls

- **Different schemas on one store.** `Retrieve` and `WriteToMemory` must name the same `schema`. A mismatch fails agent creation. A save from another schema fails `LoadGameMemory` with `8102 SchemaMismatch`.
- **`embedded_string_storage` set with `FromName`.** Leave it empty. It is for prebuilt knowledge files.
- **Writing before retrieving.** If `remember` runs first, the reply still does not see the new line this turn. Keep `Retrieve` first.
- **Load fails with `StorageBusy`.** Wait until every agent is idle. Finish or cancel open turns, and resume or cancel paused ones.

## Related

- [Reference: WriteToMemory node](https://docs.tryllengine.com/v0.10/reference/nodes/write-to-memory.html)
- [Reference: Retrieve node](https://docs.tryllengine.com/v0.10/reference/nodes/retrieve.html)
- [Reference: Error codes, 8xxx](https://docs.tryllengine.com/v0.10/reference/error-codes.html#8xxx-embedded-string-storage)
- [How-to: Inspect and Edit Knowledge Stores](https://docs.tryllengine.com/v0.10/how-to/inspect-knowledge-stores.html)
- [How-to: Save and Load Agent State](https://docs.tryllengine.com/v0.10/how-to/save-and-load-agent-state.html)
- [How-to: Use Mustache Templates](https://docs.tryllengine.com/v0.10/how-to/use-mustache-templates.html)
- [Use case: Talking NPC, memory and summarization](https://docs.tryllengine.com/v0.10/use-cases/talking-npc.html#memory-and-summarization)
