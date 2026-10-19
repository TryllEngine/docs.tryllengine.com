# Tryll

**On-device AI for games and tools.** Tryll runs small language models — and speech recognition, speech synthesis, and vector search — on the player's own machine. Instead of a single prompt call, you compose an **agent**: a workflow graph of typed nodes that retrieves from your own data, calls your game's functions, speaks and listens, and reads live game state. First-party clients for **Unity 6**, **Unreal Engine 5**, **C++**, and **Python** drive it over a binary protocol. No cloud API, no per-token cost, no network round-trip.

## Start here

New to Tryll? Read **[Tryll in Ten Minutes](https://docs.tryllengine.com/v0.9/concepts/tryll-in-ten-minutes.html)** first — the whole idea in one sitting. Then pick your engine:

- **Unity 6**

  Drop a `TryllAgentComponent` on a GameObject and chat with it.

  [First inference in Unity →](https://docs.tryllengine.com/v0.9/getting-started/first-inference-unity.html)

- **Unreal Engine 5**

  Drop a `UTryllAgentComponent` on an actor and chat with it.

  [First inference in Unreal →](https://docs.tryllengine.com/v0.9/getting-started/first-inference-unreal.html)

- **Python**

  Pip-install the client and see streaming tokens in 20 lines.

  [First inference in Python →](https://docs.tryllengine.com/v0.9/getting-started/first-inference-python.html)

- **C++**

  Clone, configure, build, and run a tiny chat demo.

  [First inference in C++ →](https://docs.tryllengine.com/v0.9/getting-started/first-inference-cpp.html)

## What Tryll gives you

- **A local model host.** One server process, many clients. It loads a model once and shares it across every session that needs it, and it ships *inside* the Unity package and Unreal plugin — there is no separate server to install or point at. [Model management →](https://docs.tryllengine.com/v0.9/reference/model-management.html)
- **Agents are workflow graphs, not prompts.** An agent is a typed graph of [nodes](https://docs.tryllengine.com/v0.9/reference/nodes/index.html) — generate, retrieve, classify, branch, call a tool, speak — wired by named exits and passing values to each other through a per-turn [slot](https://docs.tryllengine.com/v0.9/concepts/slots-and-inter-node-value-passing.html) blackboard. [Workflows, graphs, and nodes →](https://docs.tryllengine.com/v0.9/concepts/workflows-graphs-nodes.html)
- **Your data, your rules.** Ground answers in your own lore with on-device [retrieval](https://docs.tryllengine.com/v0.9/concepts/rag.html) (dense, lexical, or hybrid), force machine-readable output with a [GBNF grammar](https://docs.tryllengine.com/v0.9/concepts/constrained-output.html), and short-circuit off-topic or out-of-character input before a model ever runs.
- **Voice both ways.** Streaming [text-to-speech](https://docs.tryllengine.com/v0.9/concepts/tts-and-voice-output.html) with zero-shot voice cloning, and streaming [speech-to-text](https://docs.tryllengine.com/v0.9/concepts/stt-and-voice-input.html) with push-to-talk or hands-free turn taking — both on-device, alongside the language model.
- **Live game state in, real actions out.** [Agent variables](https://docs.tryllengine.com/v0.9/concepts/agent-variables.html) push the player's level, mood, or inventory straight into prompts and retrieval filters, and [tool calling](https://docs.tryllengine.com/v0.9/concepts/tool-calling.html) lets the model invoke your functions and answer from the results.
- **Four first-party clients, and editor tooling.** Thin, idiomatic wrappers over the same wire protocol — [Unity](https://docs.tryllengine.com/v0.9/reference/client-api/unity/index.html) · [Unreal](https://docs.tryllengine.com/v0.9/reference/client-api/unreal/cpp/index.html) · [C++](https://docs.tryllengine.com/v0.9/reference/client-api/cpp/index.html) · [Python](https://docs.tryllengine.com/v0.9/reference/client-api/python/index.html) — plus a visual graph editor, model manager, chat window, and turn inspector inside both engines.

## Explore the docs

- **[Getting Started](https://docs.tryllengine.com/v0.9/getting-started/index.html)** — tutorials for your first inference on each platform.
- **[How-to Guides](https://docs.tryllengine.com/v0.9/how-to/index.html)** — task recipes: RAG assistants, tool calls, streaming to UI, local model setup, guardrails.
- **[Concepts](https://docs.tryllengine.com/v0.9/concepts/index.html)** — the mental model, starting with [Tryll in Ten Minutes](https://docs.tryllengine.com/v0.9/concepts/tryll-in-ten-minutes.html): architecture, agents, workflows, RAG, projection.
- **[Reference](https://docs.tryllengine.com/v0.9/reference/index.html)** — every field, every enum, every error code.

## What you can build

- **An NPC that talks in character** — and keeps doing it over a long conversation, because history is projected and trimmed for you. [Design an NPC prompt →](https://docs.tryllengine.com/v0.9/how-to/design-an-npc-prompt.html)
- **An NPC that knows your world** — quest logs, item descriptions, faction lore, indexed on-device and retrieved per line of dialogue. [Create a simple RAG assistant →](https://docs.tryllengine.com/v0.9/how-to/create-simple-rag-assistant.html)
- **An NPC that acts** — the model calls your functions (open the door, set the price, start the quest), you run them, and it answers from the real result. [Define and handle tool calls →](https://docs.tryllengine.com/v0.9/how-to/define-and-handle-tool-calls.html)
- **A conversation you can hold out loud** — the player speaks, the character answers in a cloned voice, all locally. [Add voice output →](https://docs.tryllengine.com/v0.9/how-to/add-voice-to-an-agent.html) · [Use voice input →](https://docs.tryllengine.com/v0.9/how-to/use-voice-input.html)
- **Output your game code can trust** — a grammar makes a reply parse-safe by construction, and game code can own the numbers while the model owns the wording. [Constrain output with a grammar →](https://docs.tryllengine.com/v0.9/how-to/constrain-output-with-a-grammar.html) · [Keep merchant prices deterministic →](https://docs.tryllengine.com/v0.9/how-to/keep-merchant-prices-deterministic.html)
- **A character that stays in its world** — reject jailbreaks and out-of-character prompts with scripted replies, before spending a model call. [Build an immersion guard →](https://docs.tryllengine.com/v0.9/how-to/build-an-immersion-guard.html)

## Status and versioning

Tryll is in pre-release — expect breaking changes until the 1.0 API freeze. Every release documents what breaks and how to migrate; see the [release notes](https://docs.tryllengine.com/v0.9/release-notes/index.html).

Docs are published per minor version (`/vX.Y/`). The header version switcher jumps between snapshots; an archived page shows a banner when you are not on the current release. To see which plugin you have installed, read `version` in Unity's `package.json`, `VersionName` in Unreal's `.uplugin`, or the `**Tryll plugin version:**` line in the package's `AGENTS.md`.

Tryll is currently in **closed beta**: the source repository is private and stays that way, and builds go to partners we have an agreement with. To report a problem or ask a question, email **[support@tryllengine.com](mailto:support@tryllengine.com)** — please include your Tryll version, engine and version, and the relevant server log.
