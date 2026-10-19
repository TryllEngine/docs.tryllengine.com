# Tryll in Ten Minutes

This page is the whole idea of Tryll in one sitting: what it is, why it runs where it runs, and the handful of building blocks everything else is made of. Read it before anything else in these docs — every tutorial, how-to guide, and concept page assumes the picture drawn here. Each section ends with a link to the page that goes deeper.

## What Tryll is

Tryll runs **small language models** — together with speech recognition, speech synthesis, and vector search — **on the player's own PC**, inside your Unity or Unreal game. It ships inside the Unity package and the Unreal plugin and starts itself; there is no service to host and nothing to call over the internet.

Running a model is the easy part. The hard part is getting a small model to behave like *your* character, in *your* world, following *your* rules. So Tryll is mostly a **harness** around the model: the scaffolding that decides what the model sees, what it is allowed to say, and what happens with its output. If you know how a coding agent wraps a large model in tools and loops, or what a LangGraph-style workflow is, this is the same idea — built for small local models inside a game.

## Why on the player's machine

It is a trade-off, and it is worth being clear about both sides.

**What you gain:**

- **No bill that grows with your success.** A cloud model charges per token, so every extra player and every extra hour of play costs you money. On-device, the player's hardware does the work.
- **No credits to abuse.** When a player talks your NPC into writing them a Java program, that is their GPU's time, not your invoice.
- **Privacy and reliability.** Nothing the player says leaves the machine. It works offline, with no network round-trip and no outage that is not on the player's desk.

**What it costs you:**

- **You share the machine with your game.** The model needs GPU memory and RAM that your renderer would also like to have.
- **Small models need structure.** A 3–4 billion parameter model will not hold a huge lore dump in its head or improvise reliably from a vague prompt. It does well when each step it takes is small and well defined — which is exactly what the rest of this page is about.

→ [What your machine needs](https://docs.tryllengine.com/v0.9/getting-started/index.html#what-your-machine-needs) · [Estimate memory footprint](https://docs.tryllengine.com/v0.9/how-to/estimate-memory-footprint.html)

## The model is the small part: agents are workflows

An **agent** is one character, or one conversation. Its behaviour is not a prompt; it is a **workflow graph** of small steps called **nodes**, which the agent walks once every time the player says something. Some nodes call a model. Most do not.

Here is the simplest useful agent, the *Safe Receptionist* from the demo project. It has two jobs: answer the player, and refuse what it should not answer.

```
flowchart LR
    P([Player line]) --> G{guard<br>Regex Guardrail}
    G -- triggered --> D[deflect<br>Canned Response]
    G -- not triggered --> A[answer<br>Generate]
    D --> E([Reply])
    A --> E
```

The guard checks the line against a list of patterns. If one matches, a scripted reply goes back and **no model runs at all**. Only a clean line reaches `answer`, the one node that calls the language model. Real agents add more steps — retrieve some lore, classify the request, call a game function — but they are all built the same way.

This is the key design choice: **the rules and the decisions stay in your game.** The model supplies words and choices, inside boundaries you set. Whether the door opens, how much the sword costs, and when the killer's name may be revealed are for your game code to decide.

→ [Workflows, graphs, and nodes](https://docs.tryllengine.com/v0.9/concepts/workflows-graphs-nodes.html) · [All node types](https://docs.tryllengine.com/v0.9/reference/nodes/index.html)

## The building blocks

Every Tryll feature is one of the following, placed as a node in the graph or as data the graph reads.

**The character and the conversation.** The system prompt defines the character: who they are, what world they live in, how they talk, plus a few example lines. It is always kept. As a conversation grows, the oldest turns are trimmed so the prompt stays within budget — the character never forgets who it is, but it does eventually forget what you said an hour ago. → [Design an NPC prompt](https://docs.tryllengine.com/v0.9/how-to/design-an-npc-prompt.html) · [Projection and token budgets](https://docs.tryllengine.com/v0.9/concepts/projection-and-token-budgets.html)

**Knowledge.** Do not paste all your lore into the prompt. Keep it as a knowledge base of short facts, and let a Retrieve node pick the few that are relevant to each line of dialogue (this is *retrieval-augmented generation*, or RAG). It can search by meaning, by exact keywords — useful for the made-up words in your setting — or both. Facts can carry metadata, so you can gate them: the detective only learns what the player has already uncovered. → [Retrieval-augmented generation](https://docs.tryllengine.com/v0.9/concepts/rag.html) · [Create a simple RAG assistant](https://docs.tryllengine.com/v0.9/how-to/create-simple-rag-assistant.html)

**Game state in.** Agent variables let your game push live values — the player's level, the NPC's mood, the current quest stage — straight into prompts and knowledge filters, so the character reacts to what is actually happening. → [Agent variables](https://docs.tryllengine.com/v0.9/concepts/agent-variables.html) · [Drive prompts and retrieval from game state](https://docs.tryllengine.com/v0.9/how-to/drive-prompts-and-retrieval-from-game-state.html)

**Decisions.** A classifier node works out what the player *means* — a threat, a bribe, a question about the quest — and the graph branches on the answer. The same technique can sort a line by tone, which is how you move an NPC's attitude towards the player. → [Build an intent-driven NPC](https://docs.tryllengine.com/v0.9/how-to/build-intent-driven-npc.html) · [Classify intent without a knowledge base](https://docs.tryllengine.com/v0.9/how-to/classify-intent-without-a-knowledge-base.html) · [Branch on a slot](https://docs.tryllengine.com/v0.9/how-to/branch-on-a-slot.html)

**Actions out.** With a tool call, the model picks an action and its arguments — *open the gate*, *follow the player*, *sell item 12*. Your code runs it and hands back the result, and the model answers from what really happened. Tryll never runs game code itself. When you need output in an exact shape, a grammar makes anything else impossible to generate; when numbers matter, your code owns the number and the model only owns the wording. → [Tool calling](https://docs.tryllengine.com/v0.9/concepts/tool-calling.html) · [Define and handle tool calls](https://docs.tryllengine.com/v0.9/how-to/define-and-handle-tool-calls.html) · [Keep merchant prices deterministic](https://docs.tryllengine.com/v0.9/how-to/keep-merchant-prices-deterministic.html)

**Staying in character.** Pattern and intent guards, paired with scripted replies, catch jailbreaks and out-of-world questions ("what model are you?") before a model ever sees them — and can keep them out of the conversation history, so they never become context. → [Use canned responses and guardrails](https://docs.tryllengine.com/v0.9/how-to/use-canned-responses-and-guardrails.html) · [Build an immersion guard](https://docs.tryllengine.com/v0.9/how-to/build-an-immersion-guard.html)

**Voice.** The player can speak, push-to-talk or hands-free. The character can answer out loud: speech is synthesized sentence by sentence while the reply is still being written, so it starts talking almost at once. Voices can be cloned from a short sample or described in words, and lip sync drives your character's face from the same audio. → [Voice output](https://docs.tryllengine.com/v0.9/concepts/tts-and-voice-output.html) · [Voice input](https://docs.tryllengine.com/v0.9/concepts/stt-and-voice-input.html) · [Add lip sync to a character](https://docs.tryllengine.com/v0.9/how-to/add-lip-sync-to-a-character.html)

**More than one agent.** Agents are cheap next to models, so do not force one agent to do everything. Next to the character you can run a narrator, a note-taker that summarises what happened, or a judge that scores the player's argument. An agent can also be *stateless* — no memory between calls — which suits classification and other utility jobs. → [Agents and sessions](https://docs.tryllengine.com/v0.9/concepts/agents-and-sessions.html) · [Run many agents on a small GPU](https://docs.tryllengine.com/v0.9/how-to/run-many-agents-on-a-small-gpu.html)

## Models and memory

Models are files. You pick them from Tryll's catalog — or bring your own GGUF file — and download them in the editor's **Model Manager**. The models you register for production are packaged with your build, so a shipped game never downloads anything behind the player's back.

A model is loaded once and shared by every agent that uses it, so ten NPCs on the same model cost one model, plus a small working cache for each node that generates. A few numbers to calibrate against:

- **8 GB of VRAM** is the comfortable target for a game with a language model in it; 6 GB works if you stay around 3B models.
- A **3–4B model** takes roughly 2–3 GB. The tutorials use Gemma 3 4B, with Llama 3.2 3B as the smaller, quicker choice.
- **Speech recognition and the lighter voices run on the CPU** and cost no VRAM. The most expressive voices are themselves 3B models and are budgeted like one.

Each model can be placed in VRAM or in system RAM, so you can keep the GPU for rendering when the game needs it.

→ [Models and inference engines](https://docs.tryllengine.com/v0.9/concepts/models-and-inference-engines.html) · [Manage models in the editor](https://docs.tryllengine.com/v0.9/how-to/manage-models-in-editor.html) · [Choose where models run](https://docs.tryllengine.com/v0.9/how-to/choose-where-models-run.html)

## How you work with it

On a team, the work splits naturally. **Programmers** wire agents into the game: which tools exist, which variables the game pushes, what happens when the model asks to act. **Designers** shape the character: the prompt, the lore, the graph — and they can do most of that without a programmer, because Tryll lives inside the editor:

- a **graph editor** to build workflows visually,
- a **Chat** window to talk to an agent without entering Play mode,
- the **Dialog Lab** to run the same dialog across many prompt, model, and parameter variants and compare the spread — so you test changes, or a model swap, on hundreds of runs instead of a few typed lines,
- the **Agent Log** to see what each agent received and produced, and the **Memory** window to see what it all costs.

AI coding agents can help too: both plugins ship an `AGENTS.md` that briefs them on Tryll, and on Unreal they can author workflows directly through the editor's MCP server.

→ [Test an agent in the editor](https://docs.tryllengine.com/v0.9/how-to/test-agent-in-editor.html) · [Compare dialog variants](https://docs.tryllengine.com/v0.9/how-to/compare-dialog-variants.html) · [Agentic coding with Tryll](https://docs.tryllengine.com/v0.9/how-to/agentic-coding-with-tryll.html)

## What Tryll is not

- **Not a cloud API.** Everything runs locally; there is no hosted fallback.
- **Not a model.** Tryll runs open models you choose; it does not train or fine-tune them.
- **Not an image, 3D, or animation generator.** Tryll covers text, speech, and retrieval — the workloads that run well on today's player hardware.
- **Not a code runner.** The model can *ask* for an action; only your code performs it.
- **Windows only, for now.** Tryll ships as Windows 10/11 x64 binaries.

## Where to go next

| I want my character to…                | Start here                                                                                                                                                                 |
| -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Talk in character                      | [Design an NPC prompt](https://docs.tryllengine.com/v0.9/how-to/design-an-npc-prompt.html)                                                                                 |
| Know my world                          | [Create a simple RAG assistant](https://docs.tryllengine.com/v0.9/how-to/create-simple-rag-assistant.html)                                                                 |
| React to what is happening in the game | [Drive prompts and retrieval from game state](https://docs.tryllengine.com/v0.9/how-to/drive-prompts-and-retrieval-from-game-state.html)                                   |
| Understand what the player wants       | [Build an intent-driven NPC](https://docs.tryllengine.com/v0.9/how-to/build-intent-driven-npc.html)                                                                        |
| Do things in the world                 | [Define and handle tool calls](https://docs.tryllengine.com/v0.9/how-to/define-and-handle-tool-calls.html)                                                                 |
| Stay in its world                      | [Build an immersion guard](https://docs.tryllengine.com/v0.9/how-to/build-an-immersion-guard.html)                                                                         |
| Speak and listen                       | [Add voice output](https://docs.tryllengine.com/v0.9/how-to/add-voice-to-an-agent.html) · [Use voice input](https://docs.tryllengine.com/v0.9/how-to/use-voice-input.html) |
| Hold up before I ship                  | [Compare dialog variants](https://docs.tryllengine.com/v0.9/how-to/compare-dialog-variants.html)                                                                           |

Ready to try it? [Getting Started](https://docs.tryllengine.com/v0.9/getting-started/index.html) gets a model answering in your engine in about fifteen minutes. Engineers who want the process-level picture — server, sessions, and what happens on the hot path of a turn — should read [Architecture at a Glance](https://docs.tryllengine.com/v0.9/concepts/architecture-at-a-glance.html) next.

## Related

- [Reference: Workflow Nodes](https://docs.tryllengine.com/v0.9/reference/nodes/index.html)
- [Reference: Glossary](https://docs.tryllengine.com/v0.9/reference/glossary.html)
- [Concept: Architecture at a Glance](https://docs.tryllengine.com/v0.9/concepts/architecture-at-a-glance.html)
- [Concept: Workflows, Graphs, and Nodes](https://docs.tryllengine.com/v0.9/concepts/workflows-graphs-nodes.html)
- [How-to: Build a chat agent with a graph](https://docs.tryllengine.com/v0.9/how-to/build-chat-agent-with-graph.html)
