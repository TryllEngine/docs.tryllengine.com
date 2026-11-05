# Tryll

**On-device AI for games and tools.** Tryll runs small language models — and speech recognition, speech synthesis, and vector search — on the player's own machine. Instead of a single prompt call, you compose an **agent**: a workflow graph of typed nodes that retrieves from your own data, calls your game's functions, speaks and listens, and reads live game state. First-party plugins for **Unity 6** and **Unreal Engine 5** drive it over a binary protocol. No cloud API, no per-token cost, no network round-trip.

## Start here

New to Tryll? Read **[Tryll in Ten Minutes](https://docs.tryllengine.com/v0.10/concepts/tryll-in-ten-minutes.html)** first — the whole idea in one sitting.

Then browse the **[Use Cases](https://docs.tryllengine.com/v0.10/use-cases/index.html)**: talking NPCs, an ambient world, voice control, an in-game assistant. If you already have a game in mind, find the closest one and see which building blocks it takes. If you just want to make *something* with AI, skim them for an idea that stands out.

When you are ready to build, pick your engine:

- **Unity 6**

  Drop a `TryllAgentComponent` on a GameObject and chat with it.

  [First inference in Unity →](https://docs.tryllengine.com/v0.10/getting-started/first-inference-unity.html)

- **Unreal Engine 5**

  Drop a `UTryllAgentComponent` on an actor and chat with it.

  [First inference in Unreal →](https://docs.tryllengine.com/v0.10/getting-started/first-inference-unreal.html)

## What Tryll gives you

- **A local model host.** One server process, many clients. It loads a model once and shares it across every session that needs it, and it ships *inside* the Unity package and Unreal plugin — there is no separate server to install or point at. [Model management →](https://docs.tryllengine.com/v0.10/reference/model-management.html)
- **Agents are workflow graphs, not prompts.** An agent is a typed graph of [nodes](https://docs.tryllengine.com/v0.10/reference/nodes/index.html) — generate, retrieve, classify, branch, call a tool, speak — wired by named exits and passing values to each other through a per-turn [slot](https://docs.tryllengine.com/v0.10/concepts/slots-and-inter-node-value-passing.html) blackboard. [Workflows, graphs, and nodes →](https://docs.tryllengine.com/v0.10/concepts/workflows-graphs-nodes.html)
- **Your data, your rules.** Ground answers in your own lore with on-device [retrieval](https://docs.tryllengine.com/v0.10/concepts/rag.html) (dense, lexical, or hybrid), force machine-readable output with a [GBNF grammar](https://docs.tryllengine.com/v0.10/concepts/constrained-output.html), and short-circuit off-topic or out-of-character input before a model ever runs.
- **Voice both ways.** Streaming [text-to-speech](https://docs.tryllengine.com/v0.10/concepts/tts-and-voice-output.html) with zero-shot voice cloning, and streaming [speech-to-text](https://docs.tryllengine.com/v0.10/concepts/stt-and-voice-input.html) with push-to-talk or hands-free turn taking — both on-device, alongside the language model.
- **Live game state in, real actions out.** [Agent variables](https://docs.tryllengine.com/v0.10/concepts/agent-variables.html) push the player's level, mood, or inventory straight into prompts and retrieval filters, and [tool calling](https://docs.tryllengine.com/v0.10/concepts/tool-calling.html) lets the model invoke your functions and answer from the results.
- **Conversations that last.** Older turns fold into a rolling summary, so a character remembers the start of a long talk at a fixed token cost. You can seed or rewrite history, and save an agent's state with your save game and restore it later. [Keep long conversations coherent →](https://docs.tryllengine.com/v0.10/how-to/keep-long-conversations-coherent.html) · [Save and load agent state →](https://docs.tryllengine.com/v0.10/how-to/save-and-load-agent-state.html)
- **A whole cast on one GPU.** Agents share a loaded model, each node's context window is sized on its own, and agents nobody is talking to can drop out of video memory until they are needed. [Run many agents on a small GPU →](https://docs.tryllengine.com/v0.10/how-to/run-many-agents-on-a-small-gpu.html)
- **Unity and Unreal plugins, with editor tooling.** Idiomatic [Unity](https://docs.tryllengine.com/v0.10/reference/client-api/unity/index.html) and [Unreal](https://docs.tryllengine.com/v0.10/reference/client-api/unreal/cpp/index.html) APIs, plus tools inside both editors: a visual graph editor, a model manager, a chat window to test an agent without entering Play mode, a [turn inspector](https://docs.tryllengine.com/v0.10/reference/turn-inspector.html) that shows what each node did, the [Agent Log](https://docs.tryllengine.com/v0.10/how-to/use-agent-log.html), and the [Dialog Lab](https://docs.tryllengine.com/v0.10/how-to/compare-dialog-variants.html), which runs one scripted dialog across parameter variants and seeds, shows the results side by side, and also [runs in CI](https://docs.tryllengine.com/v0.10/how-to/run-dialog-lab-in-ci.html).
- **Models picked on evidence.** [Model Benchmarks](https://docs.tryllengine.com/v0.10/model-benchmarks/index.html) measure each catalog model on Tryll's own jobs (speed, video memory, and how well it guards immersion), so you can choose one for your hardware before you download it.
- **Built for coding agents.** Every package ships an `AGENTS.md`, and the docs publish an `llms.txt` index, so Claude Code, Cursor, or Copilot can write your workflows and wire them into your game. [Agentic coding with Tryll →](https://docs.tryllengine.com/v0.10/how-to/agentic-coding-with-tryll.html)

## Explore the docs

- **[Getting Started](https://docs.tryllengine.com/v0.10/getting-started/index.html)** — tutorials for your first inference in Unity and Unreal.
- **[Use Cases](https://docs.tryllengine.com/v0.10/use-cases/index.html)** — what you can build, starting from what the player experiences, and which building blocks each one takes.
- **[How-to Guides](https://docs.tryllengine.com/v0.10/how-to/index.html)** — task recipes: RAG assistants, tool calls, streaming to UI, local model setup, guardrails.
- **[Concepts](https://docs.tryllengine.com/v0.10/concepts/index.html)** — the mental model, starting with [Tryll in Ten Minutes](https://docs.tryllengine.com/v0.10/concepts/tryll-in-ten-minutes.html): architecture, agents, workflows, RAG, dialog history.
- **[Reference](https://docs.tryllengine.com/v0.10/reference/index.html)** — every field, every enum, every error code.
- **[Release Notes](https://docs.tryllengine.com/v0.10/release-notes/index.html)** — what changed in each version, and how to migrate across breaking changes.
- **[Model Benchmarks](https://docs.tryllengine.com/v0.10/model-benchmarks/index.html)** — measured speed, VRAM, and guard quality per model, to help you pick one.

## What you can build

- **An NPC that talks in character** — and keeps doing it over a long conversation, because old turns are trimmed to fit the context window for you. [Design an NPC prompt →](https://docs.tryllengine.com/v0.10/how-to/design-an-npc-prompt.html)
- **An NPC that knows your world** — quest logs, item descriptions, faction lore, indexed on-device and retrieved per line of dialogue. [Create a simple RAG assistant →](https://docs.tryllengine.com/v0.10/how-to/create-simple-rag-assistant.html)
- **An NPC that acts** — the model calls your functions (open the door, set the price, start the quest), you run them, and it answers from the real result. [Define and handle tool calls →](https://docs.tryllengine.com/v0.10/how-to/define-and-handle-tool-calls.html)
- **A conversation you can hold out loud** — the player speaks, the character answers in a cloned voice, all locally. [Add voice output →](https://docs.tryllengine.com/v0.10/how-to/add-voice-to-an-agent.html) · [Use voice input →](https://docs.tryllengine.com/v0.10/how-to/use-voice-input.html)
- **Output your game code can trust** — a grammar makes a reply parse-safe by construction, and game code can own the numbers while the model owns the wording. [Constrain output with a grammar →](https://docs.tryllengine.com/v0.10/how-to/constrain-output-with-a-grammar.html) · [Keep merchant prices deterministic →](https://docs.tryllengine.com/v0.10/how-to/keep-merchant-prices-deterministic.html)
- **A character that stays in its world** — reject jailbreaks and out-of-character prompts with scripted replies, before spending a model call. [Build an immersion guard →](https://docs.tryllengine.com/v0.10/how-to/build-an-immersion-guard.html)

## Get help

- **Ask a question** on the [Tryll Discord](https://discord.com/invite/DTKycjPpzs). It is the fastest way to reach the team and other Tryll developers.
- **Report a bug** by following [Report a bug](https://docs.tryllengine.com/v0.10/how-to/report-a-bug.html). It says which logs and details each kind of bug needs, and where to post the report.
- **Email** works too, but it is the slowest route: [support@tryllengine.com](mailto:support@tryllengine.com).
