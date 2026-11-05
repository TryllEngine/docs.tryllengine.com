# In-Game Assistant

When your game needs a helper who answers the player's questions about the game and about the current playthrough, you build an in-game assistant. It can be a ship's computer, a familiar on the player's shoulder, a guild clerk or a tutorial companion. It is a [Talking NPC](https://docs.tryllengine.com/v0.10/use-cases/talking-npc.html) whose main job is knowledge instead of drama, and it can also do things for the player.

The player asks it questions such as "what does this rune do?", "where did I leave the cart?" or "what did the mayor promise me?". The assistant answers from what it has on the game and on this playthrough.

**Building blocks:** speech-to-text, a small language model, text-to-speech, retrieval and tool calls.

______________________________________________________________________

## Two kinds of knowledge

An assistant answers from two sources, and each needs its own tool.

|               | Game knowledge                                            | Playthrough history                                              |
| ------------- | --------------------------------------------------------- | ---------------------------------------------------------------- |
| **Examples**  | Item stats, crafting recipes, lore, quest hints, controls | Where the player went, what they bought, who they promised what  |
| **Changes**   | Only when you patch the game                              | Every minute of play                                             |
| **Lives in**  | A knowledge base you ship with the game                   | Your game's own records, or a knowledge base written during play |
| **Read with** | Retrieval                                                 | Tool calls, or retrieval from a writable knowledge base          |

### Game knowledge: retrieval

Ship your game's reference material as a [knowledge base](https://docs.tryllengine.com/v0.10/concepts/rag.html) and put a [`Retrieve`](https://docs.tryllengine.com/v0.10/reference/nodes/retrieve.html) node in front of the answer. For each question, `Retrieve` adds the few matching entries to the prompt. If nothing matches, the assistant says it does not know instead of guessing.

Two rules matter most for an assistant:

- **No spoilers.** Tag each entry with the chapter or quest it belongs to, and filter on the player's progress, kept in an [agent variable](https://docs.tryllengine.com/v0.10/concepts/agent-variables.html). The assistant cannot reveal what it cannot retrieve.
- **Vague questions.** Players ask questions such as "how do I beat him?". A [query-rewriting step](https://docs.tryllengine.com/v0.10/how-to/query-rewriting-for-rag.html) combines the question with the recent conversation into a complete search query.

### Playthrough history: two options

**Option A: query your game's records with tool calls.** Your game already records what happened in its inventory, quest log, visited places and combat log. Give the assistant a tool such as `query_log(event_type, place, character)`. When the player asks "where did I last see the orc?", the model calls the tool and the turn pauses. Your game runs the query and returns the result, and the assistant answers from it.

```
flowchart LR
    Q["'Where did I last<br/>see the orc?'"] --> TC["ToolCall:<br/>query_log(...)"]
    TC -->|turn pauses| Game["Your game<br/>runs the query"]
    Game -->|result| G["GenerateAndSpeak:<br/>'In the north mine,<br/>an hour ago.'"]
```

The facts stay exact, because they come from your game's own records. Choose the model carefully, because this is the hardest of our tool-call suites. The [game-log suite](https://docs.tryllengine.com/v0.10/how-to/compare-tool-call-models.html) has two tools, four filters, and players who never name every detail. Ministral 3 14B and Qwen 3.5 4B pass 19 of 20 cases, and Llama 3.1 8B passes 12.

**Option B: a memory the assistant writes and reads.** Tryll can keep a *writable* knowledge base for the session. During play, your game or the assistant writes notes into it, such as "The mayor promised 200 gold for the wolf pelts". When the player asks, a `Retrieve` node finds the relevant notes. The [`WriteToMemory`](https://docs.tryllengine.com/v0.10/reference/nodes/write-to-memory.html) node lets the assistant record facts from the conversation with no game code. The whole memory is saved and loaded with the player's save game.

Option B suits loose, conversational facts that your game does not track in a table. Option A suits anything your game already stores precisely. Many assistants use both.

______________________________________________________________________

## Acting for the player

The same tool calls let the assistant act for the player. It can mark a location on the map, sort the inventory, set a waypoint, or pause the game and open the right menu. The model only *asks* for an action. Your game checks the request and performs the action. For anything the player might regret, such as selling an item, your game should ask the player to confirm first.

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

Keep the assistant inside the fiction where your game has one. A ship's computer that answers "as an AI language model..." breaks it. The [immersion guard](https://docs.tryllengine.com/v0.10/how-to/build-an-immersion-guard.html) from the Talking NPC page works here too.

**In the demo:** [Aquarium Expert](https://docs.tryllengine.com/v0.10/reference/demos.html#aquarium-expert) (an assistant that answers from a knowledge base, with query rewriting and an "I don't know" path), *Mother* (an experimental ship computer that reports status and reads and writes the ship's log through tools and a writable knowledge base) and *Nessa* (an experimental character who records what the player tells her with `WriteToMemory`).

## Related

- [Use Cases](https://docs.tryllengine.com/v0.10/use-cases/index.html)
- [Create a simple RAG assistant](https://docs.tryllengine.com/v0.10/how-to/create-simple-rag-assistant.html)
- [Define and handle tool calls](https://docs.tryllengine.com/v0.10/how-to/define-and-handle-tool-calls.html)
- [Design a tool-call prompt](https://docs.tryllengine.com/v0.10/how-to/design-a-tool-call-prompt.html)
- [Save and load agent state](https://docs.tryllengine.com/v0.10/how-to/save-and-load-agent-state.html)
- [Drive prompts and retrieval from game state](https://docs.tryllengine.com/v0.10/how-to/drive-prompts-and-retrieval-from-game-state.html)
