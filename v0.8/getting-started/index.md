# Getting Started

In about fifteen minutes you will install Tryll, download a model, and stream a reply from a language model running on your own machine. Pick the client that matches your project:

- **[First inference in Unity](https://docs.tryllengine.com/v0.8/getting-started/first-inference-unity.html)** Install the UPM package, add `TryllAgentComponent` to a GameObject, see streaming text in the Console. The server is bundled and launches itself.
- **[First inference in Unreal](https://docs.tryllengine.com/v0.8/getting-started/first-inference-unreal.html)** ★ Hero tutorial. Install the plugin, add `UTryllAgentComponent` to a sample actor, see streaming text in the Output Log. The server is bundled and launches itself.
- **[First inference in Python](https://docs.tryllengine.com/v0.8/getting-started/first-inference-python.html)** `pip install tryll-client`, a 20-line script, streamed tokens on stdout. You point it at a server yourself — see [Run the Tryll Server](https://docs.tryllengine.com/v0.8/how-to/run-the-tryll-server.html).
- **[First inference in C++](https://docs.tryllengine.com/v0.8/getting-started/first-inference-cpp.html)** Build the client library with CMake and run the `test-chat` console demo.

Each tutorial starts by downloading the distribution and ends with a working agent — including acquiring the model, which Tryll never does behind your back.

## What your machine needs

Tryll runs the model on the player's hardware, so the hardware is part of the design. Check this before you start.

**Operating system**

- **Windows 10 or 11, 64-bit.** Tryll ships as Windows x64 binaries — the server, the Unity package, the Unreal plugin, and the C++ client library. There is no Linux or macOS distribution.

**GPU**

- **8 GB of VRAM or more is the comfortable target.** That fits a quantised 7–8B model with a useful context window, and leaves room for speech synthesis and recognition running alongside it.
- **6 GB works** if you stay around **3B** models — a good fit for focused NPC dialogue, intent classification, and routing.
- **More VRAM is mostly about freedom to experiment**: larger models, longer contexts, and several agents resident at once. It is not required to ship something good.
- Speech-to-text and text-to-speech run on the **CPU** and cost no VRAM.

**Disk and RAM**

- **1.5–5 GB per language model**, plus a working internet connection the first time you download one.
- Budget **roughly the same again in system RAM**: a model file is memory-mapped *and* offloaded to the GPU, so it counts in both columns.

Work out the real numbers for a specific line-up with [Estimate the RAM and VRAM your setup needs](https://docs.tryllengine.com/v0.8/how-to/estimate-memory-footprint.html).

**Engine and language versions**

| Path   | Needs                                                                                    |
| ------ | ---------------------------------------------------------------------------------------- |
| Unity  | Unity 6 (6000.x) or later                                                                |
| Unreal | Unreal Engine 5.7, a **C++** project (Blueprint-only projects cannot compile the plugin) |
| Python | Python 3.10+                                                                             |
| C++    | Visual Studio 2022, CMake                                                                |

## What you will build

Each tutorial ends at the same milestone: the user sends one message, the server runs it through a minimal graph (just a `Generate` node), and the client prints the streamed reply. Everything else — retrieval, tool calls, voice, guardrails — builds on top of this pattern.

## After the tutorial

You have a one-node graph. These are the shortest paths to something that feels like a game feature:

- **Author the graph visually instead of in code.** [Edit workflows in the Unity graph editor](https://docs.tryllengine.com/v0.8/how-to/edit-workflows-in-graph-editor.html) or [in the Unreal graph editor](https://docs.tryllengine.com/v0.8/how-to/edit-workflows-in-unreal-graph-editor.html)
- **Iterate without entering Play mode.** [Test an agent in the editor](https://docs.tryllengine.com/v0.8/how-to/test-agent-in-editor.html)
- **Write a character worth talking to.** [Design an NPC prompt](https://docs.tryllengine.com/v0.8/how-to/design-an-npc-prompt.html)
- **Ground answers in your own lore.** [Create a simple RAG assistant](https://docs.tryllengine.com/v0.8/how-to/create-simple-rag-assistant.html)
- **Give the character a voice.** [Add voice output to an agent](https://docs.tryllengine.com/v0.8/how-to/add-voice-to-an-agent.html)
- **Feed it live game state.** [Drive prompts and retrieval from game state](https://docs.tryllengine.com/v0.8/how-to/drive-prompts-and-retrieval-from-game-state.html)

If you want the *why* before the *how*, start with [Architecture at a Glance](https://docs.tryllengine.com/v0.8/concepts/architecture-at-a-glance.html).
