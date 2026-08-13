# Concepts

These pages explain *how Tryll thinks* — the mental models behind the server, the client libraries, and the workflows you build on top of them. Read one when you want to understand *why* something works the way it does; reach for a [how-to guide](https://docs.tryllengine.com/how-to/index.html) or the [reference](https://docs.tryllengine.com/reference/index.html) when you want to do a specific thing or look up a specific field.

## Start with these

- **[Architecture at a Glance](https://docs.tryllengine.com/concepts/architecture-at-a-glance.html)** — the 10-minute tour. Server → session → agent → graph → node → model. Every later page fits inside this frame.
- **[Workflows, Graphs, and Nodes](https://docs.tryllengine.com/concepts/workflows-graphs-nodes.html)** — the building block that turns a model into an application.
- **[Agents and Sessions](https://docs.tryllengine.com/concepts/agents-and-sessions.html)** — who owns what, and how a conversation comes into and goes out of existence.
- **[Lifetime and Ownership](https://docs.tryllengine.com/concepts/lifetime-and-ownership.html)** — how string storages, embedded string storages, and models stay alive across agents, turns, and sessions.

## Dive into the pieces

- **[Models and Inference Engines](https://docs.tryllengine.com/concepts/models-and-inference-engines.html)** — language vs. embedding models, variants per backend, what the KV cache does.
- **[TTS and Voice Output](https://docs.tryllengine.com/concepts/tts-and-voice-output.html)** — how streaming speech synthesis works: `GenerateAndSpeak` vs. `Speak`, PCM audio frames, voices, and the Supertonic / Pocket TTS families.
- **[STT and Voice Input](https://docs.tryllengine.com/concepts/stt-and-voice-input.html)** — how spoken audio becomes text: the utterance lifecycle, push-to-talk vs. hands-free, offline vs. streaming recognisers, and auto-sending into an agent.
- **[Retrieval-Augmented Generation (RAG)](https://docs.tryllengine.com/concepts/rag.html)** — how Tryll grounds answers in your data: chunking, embeddings, cosine distance, and where retrieved text lands in the prompt.
- **[Tool Calling](https://docs.tryllengine.com/concepts/tool-calling.html)** — making small local models invoke tools: chat-template-driven prompting, how results flow back into the dialogue, and the limits that matter on-device.
- **[Agent Variables](https://docs.tryllengine.com/concepts/agent-variables.html)** — a typed, per-agent key→value store the game writes over the wire and the server reads from Mustache templates (`{{var.<name>}}`), the Retrieve filter, and (opt-in) `__NAME__` markers in Generate / GenerateAndSpeak output.
- **[Slots and Inter-node Value Passing](https://docs.tryllengine.com/concepts/slots-and-inter-node-value-passing.html)** — the per-turn blackboard nodes read and write: `input` selection, `{{slot.<name>}}` templates, and `send` / `history_role` delivery.
- **[Constrained Output (GBNF)](https://docs.tryllengine.com/concepts/constrained-output.html)** — forcing a model to emit exactly a fixed shape (a command, a choice, a tiny JSON object) with a grammar: what it guarantees (syntax) and what it never does (meaning).
- **[Projection and Token Budgets](https://docs.tryllengine.com/concepts/projection-and-token-budgets.html)** — how a growing dialog is turned into a bounded prompt, turn after turn, without losing the session.

## How these relate to the rest of the docs

```
flowchart LR
    C[Concepts<br>why & mental model] --> R[Reference<br>fields & semantics]
    C --> H[How-to<br>task recipes]
    H --> R
    T[Tutorials<br>first inference] --> H
    T --> C
```

Every concept page links down to the specific [reference](https://docs.tryllengine.com/reference/index.html) pages that make its ideas concrete, and to the [how-to guides](https://docs.tryllengine.com/how-to/index.html) that put them to use.
