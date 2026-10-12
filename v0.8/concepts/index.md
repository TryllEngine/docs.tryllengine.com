# Concepts

These pages explain *how Tryll thinks* — the mental models behind the server, the client libraries, and the workflows you build on top of them. Read one when you want to understand *why* something works the way it does; reach for a [how-to guide](https://docs.tryllengine.com/v0.8/how-to/index.html) or the [reference](https://docs.tryllengine.com/v0.8/reference/index.html) when you want to do a specific thing or look up a specific field.

## Start with these

- **[Tryll in Ten Minutes](https://docs.tryllengine.com/v0.8/concepts/tryll-in-ten-minutes.html)** — read this first. What Tryll is, why it runs on the player's machine, why an agent is a workflow rather than a prompt, and the building blocks every other page is made of.
- **[Architecture at a Glance](https://docs.tryllengine.com/v0.8/concepts/architecture-at-a-glance.html)** — the engineer's view. Server → session → agent → graph → node → model, and the hot path of one turn.
- **[Workflows, Graphs, and Nodes](https://docs.tryllengine.com/v0.8/concepts/workflows-graphs-nodes.html)** — the building block that turns a model into an application.
- **[Agents and Sessions](https://docs.tryllengine.com/v0.8/concepts/agents-and-sessions.html)** — who owns what, and how a conversation comes into and goes out of existence.
- **[Lifetime and Ownership](https://docs.tryllengine.com/v0.8/concepts/lifetime-and-ownership.html)** — how string storages, embedded string storages, and models stay alive across agents, turns, and sessions.

## Dive into the pieces

- **[Models and Inference Engines](https://docs.tryllengine.com/v0.8/concepts/models-and-inference-engines.html)** — the catalog's model types (language, embedding, speech-to-text, text-to-speech, voice activity), the one engine each catalog model runs on, and what the KV cache does.
- **[TTS and Voice Output](https://docs.tryllengine.com/v0.8/concepts/tts-and-voice-output.html)** — how streaming speech synthesis works: `GenerateAndSpeak` vs. `Speak`, PCM audio frames, voices, and the Supertonic, Pocket TTS, Orpheus, and Maya1 families.
- **[STT and Voice Input](https://docs.tryllengine.com/v0.8/concepts/stt-and-voice-input.html)** — how spoken audio becomes text: the utterance lifecycle, push-to-talk vs. hands-free, and offline vs. streaming recognisers.
- **[Retrieval-Augmented Generation (RAG)](https://docs.tryllengine.com/v0.8/concepts/rag.html)** — how Tryll grounds answers in your data: chunking, embeddings, dense, lexical, and hybrid search, and where retrieved text lands in the prompt.
- **[Tool Calling](https://docs.tryllengine.com/v0.8/concepts/tool-calling.html)** — making small local models invoke tools: chat-template-driven prompting, limits, and what the detection-based design means for your game or app. To pick a catalog model or shape the schema, see [Compare tool-call models](https://docs.tryllengine.com/v0.8/how-to/compare-tool-call-models.html) and [Design a tool-call prompt](https://docs.tryllengine.com/v0.8/how-to/design-a-tool-call-prompt.html).
- **[Agent Variables](https://docs.tryllengine.com/v0.8/concepts/agent-variables.html)** — a typed, per-agent key→value store the game writes over the wire and the server reads from Mustache templates (`{{var.<name>}}`), the Retrieve filter, and (opt-in) `__NAME__` markers in Generate / GenerateAndSpeak output.
- **[Slots and Inter-node Value Passing](https://docs.tryllengine.com/v0.8/concepts/slots-and-inter-node-value-passing.html)** — the per-turn blackboard nodes read and write: `input` selection, `{{slot.<name>}}` templates, and `send` / `history_role` delivery.
- **[Constrained Output (GBNF)](https://docs.tryllengine.com/v0.8/concepts/constrained-output.html)** — forcing a model to emit exactly a fixed shape (a command, a choice, a tiny JSON object) with a grammar: what it guarantees (syntax) and what it never does (meaning).
- **[Projection and Token Budgets](https://docs.tryllengine.com/v0.8/concepts/projection-and-token-budgets.html)** — how a growing dialog is turned into a bounded prompt, how the token budget trims it, and how a rolling summary keeps the thread.

## How these relate to the rest of the docs

```
flowchart LR
    C[Concepts<br>why & mental model] --> R[Reference<br>fields & semantics]
    C --> H[How-to<br>task recipes]
    H --> R
    T[Tutorials<br>first inference] --> H
    T --> C
```

Every concept page links down to the specific [reference](https://docs.tryllengine.com/v0.8/reference/index.html) pages that make its ideas concrete, and to the [how-to guides](https://docs.tryllengine.com/v0.8/how-to/index.html) that put them to use.
