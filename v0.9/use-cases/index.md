# Use Cases

What can you build with Tryll, and which pieces do you need for it? Each page in this section starts from something a player experiences and walks through the building blocks that produce it. The pages link to the [How-to Guides](https://docs.tryllengine.com/v0.9/how-to/index.html) for the exact steps, so read a use case first to choose your design, then follow the links to build it.

All of these run on the player's own PC, inside your Unity or Unreal game.

## The building blocks

Every use case combines the same few blocks:

| Block                          | What it does for your game                                                                                     |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------- |
| **Speech-to-text (STT)**       | Turns the player's voice into text.                                                                            |
| **Small language model (SLM)** | Reads the conversation and writes the reply.                                                                   |
| **Text-to-speech (TTS)**       | Speaks the reply in a character's voice.                                                                       |
| **Retrieval (RAG)**            | Looks up lore, history or game facts and hands the relevant ones to the model.                                 |
| **Classification**             | Reads the player's line and turns it into a value your game can use, such as "accusing", "flirting" or "rude". |
| **Tool calls**                 | Lets the model ask your game to do something, such as open a door or attack. Your code decides and runs it.    |
| **Graph**                      | Connects the blocks. An agent is a graph of nodes, edited visually in Unity or Unreal.                         |

## The use cases

| Use case                                                                                    | The player experiences                                                                                                                                                            | Main blocks                                               |
| ------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| **[Talking NPC](https://docs.tryllengine.com/v0.9/use-cases/talking-npc.html)**             | A character you speak with, who knows the world, plays a mechanic, shows emotion and acts. Built up one step at a time.                                                           | STT, SLM, TTS, then retrieval, classification, tool calls |
| **[Ambient world](https://docs.tryllengine.com/v0.9/use-cases/ambient-world.html)**         | Background characters who chat among themselves, and a narrator, radio or newspaper that reports what happened in the game.                                                       | SLM, TTS, game events, background priority                |
| **[Voice control](https://docs.tryllengine.com/v0.9/use-cases/voice-control.html)**         | Commanding the game or your squad by voice. Characters may answer with prewritten lines.                                                                                          | STT, SLM, tool calls                                      |
| **[In-game assistant](https://docs.tryllengine.com/v0.9/use-cases/in-game-assistant.html)** | A helper that knows the game, remembers what you did, looks things up and acts for you.                                                                                           | STT, SLM, TTS, retrieval, tool calls                      |
| **[AI director](https://docs.tryllengine.com/v0.9/use-cases/ai-director.html)** *(idea)*    | A model that plans quests or directs the story, while your game systems carry out the plan. We have not built this yet; the page shows the research and the blocks you would use. | SLM, grammar-constrained output, tool calls               |
