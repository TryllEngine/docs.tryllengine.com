# Use Cases

If you want a character the player can talk to, or a game that takes commands by voice, these pages show which parts of Tryll to combine. Everything runs on the player's own PC, inside your Unity or Unreal game.

Each use case starts from something the player experiences and explains the building blocks that produce it. Read a use case to choose your design, then follow its links to the [How-to Guides](https://docs.tryllengine.com/v0.10/how-to/index.html) for the exact steps.

## The building blocks

Every use case combines the same few blocks:

| Block                          | What it does for your game                                                                                        |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------- |
| **Speech-to-text (STT)**       | Turns the player's voice into text.                                                                               |
| **Small language model (SLM)** | Takes the conversation so far and writes the reply.                                                               |
| **Text-to-speech (TTS)**       | Speaks the reply in a character's voice.                                                                          |
| **Retrieval (RAG)**            | Looks up lore, history or game facts and hands the relevant ones to the model.                                    |
| **Classification**             | Turns the player's line into a value your game can use, such as "accusing", "flirting" or "rude".                 |
| **Tool calls**                 | Lets the model ask your game to do something, such as open a door or attack. Your code decides whether to run it. |
| **Graph**                      | Connects the blocks. An agent is a graph of nodes that you edit visually in Unity or Unreal.                      |

## The use cases

| Use case                                                                                     | The player experiences                                                                                                                                                     | Main blocks                                                       |
| -------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| **[Talking NPC](https://docs.tryllengine.com/v0.10/use-cases/talking-npc.html)**             | A character the player talks to that stays in its role. Optional features add knowledge, game mechanics, emotion, actions and memory.                                      | STT, SLM, TTS, then retrieval, classification, tool calls, memory |
| **[Ambient world](https://docs.tryllengine.com/v0.10/use-cases/ambient-world.html)**         | Background characters who chat among themselves, and a narrator, radio or newspaper that reports what happened in the game.                                                | SLM, TTS, game events, background priority                        |
| **[Voice control](https://docs.tryllengine.com/v0.10/use-cases/voice-control.html)**         | The player commands the game or their squad by voice. Characters can answer with prewritten lines.                                                                         | STT, SLM, tool calls                                              |
| **[In-game assistant](https://docs.tryllengine.com/v0.10/use-cases/in-game-assistant.html)** | A helper that looks up facts about the game, remembers what the player did, and acts for the player.                                                                       | STT, SLM, TTS, retrieval, tool calls                              |
| **[AI director](https://docs.tryllengine.com/v0.10/use-cases/ai-director.html)** *(idea)*    | A model plans quests or directs the story, and your game systems carry out the plan. We have not built this yet; the page shows the research and the blocks you would use. | SLM, grammar-constrained output, tool calls                       |
