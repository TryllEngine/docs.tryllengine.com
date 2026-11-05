# Video Walkthroughs

Screen recordings that build and test the demo agents from the Unity demo project (`TryllUnityDemo`), one node at a time. Each video opens the workflow in the graph editor, explains the parameters that matter, talks to the agent in **Tryll Chat**, and then tests it in the **Dialog Lab**. The nodes and parameters are the same in Unreal; only the editor windows look different.

The whole series is also a YouTube playlist: [Tryll documentation videos](https://www.youtube.com/playlist?list=PLZqr8WpRovB4).

| Video                                                       | Length | What it shows                                                       |
| ----------------------------------------------------------- | ------ | ------------------------------------------------------------------- |
| [Safe Receptionist](#safe-receptionist)                     | 9:21   | Guardrails, canned responses, and testing with the Dialog Lab       |
| [Voice Character](#voice-character)                         | 4:43   | Voice output with Generate and Speak, voice input outside the graph |
| [Quest Giver](#quest-giver)                                 | 6:36   | Agent variables, the Transform node, and slots                      |
| [Aquarium Expert](#aquarium-expert)                         | 9:20   | The Branch node and simple RAG with a Retrieve node                 |
| [Aquarium Expert, advanced path](#aquarium-expert-advanced) | 13:27  | Query rewriting, HyDE, and hybrid retrieval                         |

Watch them in order: each video builds on the ones before it.

## Safe Receptionist

The simplest useful agent: a hotel receptionist that **answers** guests and **refuses** what it should not answer. Three nodes and one branch.

- **The graph.** A [Regex Guardrail](https://docs.tryllengine.com/v0.10/reference/nodes/regex-guardrail.html) node checks the player's message first. On `triggered` the turn goes to a [Canned Response](https://docs.tryllengine.com/v0.10/reference/nodes/canned-response.html) node; on `not_triggered` it goes to a [Generate](https://docs.tryllengine.com/v0.10/reference/nodes/generate.html) node.
- **The guardrail.** A short inline list of regular expressions, and when to move the list into a [string storage](https://docs.tryllengine.com/v0.10/reference/string-storage.html) file instead.
- **The refusal.** Selection Strategy, why a canned line is sent whole rather than streamed, and what **History Role** does on the *next* turn.
- **The model.** Why Generate nodes leave Model Name empty and use the workflow's Default Model Name, and why a prompt-level rule is weaker than the regex node.
- **Testing.** Why one chat window is not enough. A dialog script that checks both routes, including a **false positive** check on a harmless question. Five repeats of one variant to see the spread, then three models side by side.

Read next: [Use Canned Responses and Guardrails](https://docs.tryllengine.com/v0.10/how-to/use-canned-responses-and-guardrails.html), [Compare Dialog Variants in the Lab](https://docs.tryllengine.com/v0.10/how-to/compare-dialog-variants.html).

## Voice Character

An agent you talk to and that talks back. The whole workflow is one node.

- **Voice out.** One [Generate and Speak](https://docs.tryllengine.com/v0.10/reference/nodes/generate-and-speak.html) node. It speaks each sentence as soon as it is finished, while the model writes the next one. Min Sentence Chars trades time to first audio against more, smaller synthesis calls.
- **The voice.** The TTS model (Supertonic), picking a voice by Speaker Id, Speed, and which voice fields a fixed-voice model ignores.
- **A prompt written to be spoken.** Short plain sentences, no lists, no markdown, no stage directions.
- **The scene.** The agent component, the speaker (attached for you if you leave it empty), and the voice input component.
- **Voice in.** Speech recognition is not a node: the voice input component owns the microphone and sends the transcript to its **Target Agent** as if the player had typed it. Voice-activity detection, a hotwords file, and hands-free versus push-to-talk (the turn starts when you let go of the button).
- **What the Dialog Lab covers.** The Lab tests the *text* half of a speaking agent. Microphone, silence detection and playback still need a human listening.

Read next: [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.10/how-to/add-voice-to-an-agent.html), [Use Voice Input](https://docs.tryllengine.com/v0.10/how-to/use-voice-input.html), [Bias Voice Input with Hotwords](https://docs.tryllengine.com/v0.10/how-to/use-voice-input-hotwords.html).

## Quest Giver

One graph, one model, one system prompt, and a quest giver who treats a stranger very differently from the hero who saved the village. None of that difference lives in the graph: it comes from the game.

- **Variables.** Declaring typed [agent variables](https://docs.tryllengine.com/v0.10/concepts/agent-variables.html) on the workflow (a string, an integer, a string set), and why the declared value is only the starting value.
- **One asset, two NPCs.** Variable Overrides on the agent component, like a material and a material instance in Unity.
- **Slots.** The one-turn blackboard, and how a slot differs from a variable. See [Slots and Inter-node Value Passing](https://docs.tryllengine.com/v0.10/concepts/slots-and-inter-node-value-passing.html).
- **The Transform node.** A [Transform](https://docs.tryllengine.com/v0.10/reference/nodes/transform.html) node renders a Mustache template with the game state and the player's message into a slot. No model runs and nothing reaches the player.
- **Input.** The Generate node's **Input** reads that slot instead of the raw message, so the model always sees the question wrapped in the state of the world.
- **Testing.** Two Dialog Lab variants that differ only by agent, the two columns side by side, and the rendered prompt for each.

Read next: [Drive Prompts and Retrieval from Game State](https://docs.tryllengine.com/v0.10/how-to/drive-prompts-and-retrieval-from-game-state.html), [Use Mustache Templates in Prompts](https://docs.tryllengine.com/v0.10/how-to/use-mustache-templates.html).

## Aquarium Expert

An expert that answers from a knowledge base instead of from the model's own training: retrieval-augmented generation, the simple version. This video takes the simple path through the workflow; the next one takes the advanced path.

- **The Branch node.** A [Branch](https://docs.tryllengine.com/v0.10/reference/nodes/branch.html) node reads one boolean variable (`Advanced`) with the *is true* test and picks `then` or `else`. It runs no model and sends nothing.
- **How vector search works.** Text becomes a vector, similar meanings land close together, and the question is matched by meaning, not by keyword.
- **The knowledge base.** The config file, the records file, the embedding model, and what metadata is for. See [Embedded String Storage](https://docs.tryllengine.com/v0.10/reference/embedded-string-storage.html).
- **The Retrieve node.** The [Retrieve](https://docs.tryllengine.com/v0.10/reference/nodes/retrieve.html) node's Input, Retrieval Mode, Top K and Threshold. The threshold is a *maximum* distance, so a lower number is stricter. `found` goes on to the answer; `not_found` goes to a canned response read from a file.
- **Folding knowledge into the prompt.** A Transform node puts the retrieved chunks first and the question last.
- **Reading the results.** The route and the per-node chips in the Turn Inspector, the hit table with distances, and the prompt the model was sent.
- **Where it fails.** An off-topic question correctly misses; a loose threshold lets useless chunks through; a question about a fish that is not in the knowledge base produces a fluent answer from the model's own training. You can prove it is a hallucination because both halves are on screen.

Read next: [Create a Simple RAG Assistant](https://docs.tryllengine.com/v0.10/how-to/create-simple-rag-assistant.html), [Retrieval-Augmented Generation](https://docs.tryllengine.com/v0.10/concepts/rag.html).

## Aquarium Expert, advanced path

The same workflow with `Advanced` turned on: two extra steps that fix the two real flaws of plain vector search.

- **Flaw one.** The player's message is not written for searching: typos move the vector, and "they" or "that one" carry no meaning into an embedding.
- **Flaw two.** Your game's own words (items, places, invented species) are noise to an embedding model that has never seen them. Those need keyword matching.
- **The rewrite node.** A hidden Generate node with Send and History Role both set to None: the player never sees it and later turns never replay it, but it still reads the conversation, which is how it resolves "they".
- **Query rewriting versus HyDE.** Rewrite the question into a search query, or into a *hypothetical answer* written like the stored chunks. The passage may be wrong; it only has to land next to the right chunk. A Keywords line feeds the keyword search.
- **The settings that matter.** A small Context Size, Temperature 0, and a measured Max Tokens.
- **Hybrid retrieval.** Dense and BM25 searches, why their scores cannot be averaged, and how Reciprocal Rank Fusion combines ranks instead. In hybrid mode the threshold filters only the vector search.
- **Reading a hybrid hit table**, and a payoff turn: a misspelled question that never names the fish, simple path against advanced path.

Read next: [Query Rewriting and HyDE for RAG](https://docs.tryllengine.com/v0.10/how-to/query-rewriting-for-rag.html), [Add a Hidden Reasoning or Draft Step](https://docs.tryllengine.com/v0.10/how-to/add-a-hidden-reasoning-step.html).
