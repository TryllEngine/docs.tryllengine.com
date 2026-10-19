# In-Game Assistant

A helper that lives in the game: a ship's computer, a familiar on your shoulder, a guild clerk, a tutorial companion. The player asks it things ("what does this rune do?", "where did I leave the cart?", "what did the mayor promise me?") and it answers from what it knows about the game and about this playthrough. It can also do things for the player.

**Building blocks:** speech-to-text, a small language model, text-to-speech, retrieval and tool calls. It is a [Talking NPC](https://docs.tryllengine.com/v0.9/use-cases/talking-npc.html) whose main job is knowledge instead of drama.

______________________________________________________________________

## Two kinds of knowledge

An assistant answers from two very different sources, and each needs its own tool.

|               | Game knowledge                                            | Playthrough history                                              |
| ------------- | --------------------------------------------------------- | ---------------------------------------------------------------- |
| **Examples**  | Item stats, crafting recipes, lore, quest hints, controls | Where the player went, what they bought, who they promised what  |
| **Changes**   | Only when you patch the game                              | Every minute of play                                             |
| **Lives in**  | A knowledge base you ship with the game                   | Your game's own records, or a knowledge base written during play |
| **Read with** | Retrieval                                                 | Tool calls, or retrieval from a writable knowledge base          |

### Game knowledge: retrieval

Ship your game's reference material as a [knowledge base](https://docs.tryllengine.com/v0.9/concepts/rag.html) and put a [`Retrieve`](https://docs.tryllengine.com/v0.9/reference/nodes/retrieve.html) node in front of the answer. Each question pulls the few matching entries into the prompt. If nothing matches, the assistant says it does not know, instead of guessing.

Two things matter for an assistant in particular:

- **No spoilers.** Tag entries with the chapter or quest they belong to, and filter on the player's progress, kept in an [agent variable](https://docs.tryllengine.com/v0.9/concepts/agent-variables.html). The assistant cannot reveal what it cannot retrieve.
- **Vague questions.** Players ask "how do I beat him?". A [query-rewriting step](https://docs.tryllengine.com/v0.9/how-to/query-rewriting-for-rag.html) turns the question, together with the recent conversation, into a proper search.

### Playthrough history: two options

**Option A: query your game's records with tool calls.** Your game already tracks what happened: inventory, quest log, visited places, combat log. Give the assistant a tool such as `query_log(event_type, place, character)`. When the player asks "where did I last see the orc?", the model calls the tool, the turn pauses, your game runs the query and returns the result, and the assistant answers from it.

```
flowchart LR
    Q["'Where did I last<br/>see the orc?'"] --> TC["ToolCall:<br/>query_log(...)"]
    TC -->|turn pauses| Game["Your game<br/>runs the query"]
    Game -->|result| G["GenerateAndSpeak:<br/>'In the north mine,<br/>an hour ago.'"]
```

This keeps the facts exact, because they come straight from game state. Choose the model with care: this is the hardest of our tool-call suites. On our [game-log suite](https://docs.tryllengine.com/v0.9/how-to/compare-tool-call-models.html) (two tools, four filters, and players who never name every detail), NVIDIA Orchestrator 8B, Ministral 3 14B and Qwen 3.5 4B pass 19 of 20 cases, while Llama 3.1 8B passes 12.

**Option B: a memory the assistant writes and reads.** Tryll can keep a *writable* knowledge base for the session. Your game, or the agent itself, writes notes into it during play ("The mayor promised 200 gold for the wolf pelts"), and a `Retrieve` node finds the relevant ones when the player asks. The [`WriteToMemory`](https://docs.tryllengine.com/v0.9/reference/nodes/write-to-memory.html) node lets the agent record facts from the conversation without any game code. The whole memory is saved and loaded with the player's save game.

Option B suits loose, conversational facts that your game does not track in a table. Option A suits anything your game already stores precisely. Many assistants use both.

______________________________________________________________________

## Acting for the player

The same tool-call mechanism lets the assistant act: mark a location on the map, sort the inventory, set a waypoint, pause the game and open the right menu. As always, the model only *asks*; your game checks and performs the action. For anything the player might regret, such as selling an item, have your game ask for confirmation first.

______________________________________________________________________

## Putting it together

```
flowchart LR
    In["Player question"] --> TC["ToolCall<br/>(query_log, set_waypoint, ...)"]
    TC -->|tool_called| Game["Your game:<br/>query or act"]
    Game --> R["Retrieve game knowledge<br/>(filtered by progress)"]
    TC -->|no_tool_called| R
    R --> C["Transform:<br/>facts + question"]
    C --> G["GenerateAndSpeak"]
```

Keep the assistant inside the fiction where your game has one. A ship's computer that answers "as an AI language model..." breaks it. The [immersion guard](https://docs.tryllengine.com/v0.9/how-to/build-an-immersion-guard.html) from the Talking NPC page applies here too.

**In the demo:** *AquariumExpert* (an assistant that answers from a knowledge base, with query rewriting and an "I don't know" path), *Mother* (an experimental ship computer that reports status and reads and writes the ship's log through tools and a writable knowledge base) and *Nessa* (an experimental character who records what the player tells her with `WriteToMemory`).

## Related

- [Use Cases](https://docs.tryllengine.com/v0.9/use-cases/index.html)
- [Create a simple RAG assistant](https://docs.tryllengine.com/v0.9/how-to/create-simple-rag-assistant.html)
- [Define and handle tool calls](https://docs.tryllengine.com/v0.9/how-to/define-and-handle-tool-calls.html)
- [Design a tool-call prompt](https://docs.tryllengine.com/v0.9/how-to/design-a-tool-call-prompt.html)
- [Save and load agent state](https://docs.tryllengine.com/v0.9/how-to/save-and-load-agent-state.html)
- [Drive prompts and retrieval from game state](https://docs.tryllengine.com/v0.9/how-to/drive-prompts-and-retrieval-from-game-state.html)
