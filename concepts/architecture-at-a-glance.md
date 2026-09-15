# Architecture at a Glance

Tryll is a C++ TCP server that runs small language models on the user's own machine and exposes them to games and apps through a binary protocol. This page is the ten-minute mental model: what runs where, what owns what, and what the hot path looks like when a user hits "send".

Every other concept page zooms into one layer of this picture. Read this one first.

## The five layers

```
flowchart TB
    subgraph Client["Client process (your game / app / script)"]
        CL["tryll-client<br>(C++ / Python / Unreal)"]
    end
    subgraph Server["tryll-server process"]
        S[Session<br>one per TCP connection]
        A[Agent<br>one per CreateAgent]
        G[Graph<br>nodes + exit fields]
        M[Model pool<br>shared models &amp; KV caches]
    end
    subgraph Hardware["Local hardware"]
        HW[CPU / GPU<br>llama.cpp · ONNX · WindowsML]
    end
    CL -- "TCP :9100<br>FlatBuffers framing" --> S
    S --> A
    A --> G
    G --> M
    M --> HW
```

- **The server** is a single C++ process. One instance, many clients. It runs small language models on-device through [llama.cpp](https://github.com/ggerganov/llama.cpp) and uses [USearch](https://github.com/unum-cloud/usearch) for vector search; everything else — the workflow engine, the wire protocol, logging — is built inside the server.
- **A [session](https://docs.tryllengine.com/reference/glossary.html#session)** is one TCP connection. It owns the client's [agents](https://docs.tryllengine.com/reference/glossary.html#agent), [string storages](https://docs.tryllengine.com/reference/string-storage.html), and [embedded string storages](https://docs.tryllengine.com/reference/embedded-string-storage.html). When the socket closes, everything in the session goes with it.
- **An [agent](https://docs.tryllengine.com/reference/glossary.html#agent)** is one conversation. It carries its own [workflow graph](https://docs.tryllengine.com/reference/glossary.html#graph), its own dialog history, and its own default model. A single session can own several agents in parallel.
- **The [graph](https://docs.tryllengine.com/reference/glossary.html#graph)** is a set of [nodes](https://docs.tryllengine.com/reference/nodes/index.html), each owning its own exit fields that wire it to the next node. The agent walks it once per user turn — see [Workflows, Graphs, and Nodes](https://docs.tryllengine.com/concepts/workflows-graphs-nodes.html).
- **Models** are loaded once per process by the server and shared across every agent that needs them. The KV cache, by contrast, is per-node, because each node sees a different slice of the dialog.

## One turn end to end

The hot path when a user sends a message:

```
sequenceDiagram
    participant C as Client
    participant S as Server
    participant A as Agent
    participant N as Node (e.g. Generate)
    participant M as Model

    C->>S: SendMessageRequest(text)
    S->>A: dispatch to the agent
    A->>N: execute current node
    N->>M: generate / stream tokens
    M-->>N: tokens
    N-->>S: AnswerText(delta, is_final=false)
    S-->>C: AnswerText frame
    Note over N,M: repeats until stop
    N-->>A: exit route (default / found / …)
    A->>A: follow route to next node or END
    A-->>S: TurnComplete(status)
    S-->>C: TurnComplete frame
```

A turn is always atomic at the agent level. A second `SendMessage` for the same agent while a turn is live is rejected with [3001](https://docs.tryllengine.com/reference/error-codes.html#3xxx-agent); in-flight work keeps running.

## Concurrency, in one line

Turns on one session run serially — if agent A is mid-turn, agent B on the same session waits. Inference is serialised across the whole server, because there is only one GPU / CPU to talk to. Non-inference work (routing, retrieval, pattern checks) can happen in parallel.

## The wire protocol

Clients speak to the server over TCP (default port 9100) using length-prefixed FlatBuffers messages. The shape every message takes — and the end-to-end session flow — is documented in [Wire Protocol](https://docs.tryllengine.com/reference/wire-protocol.html).

The session lifecycle, in one line:

```
ConnectionReady → CreateSession → (CreateStringStorage…)* → CreateAgent → SendMessage …
```

The first three client libraries (C++, Python, Unreal) are thin wrappers over this protocol. Nothing else special happens on the client side — the intelligence lives in the server.

## Why it is shaped this way

Three product constraints drove the split:

- **On-device.** A game or tool that wants a local model cannot afford a Python runtime or a separate inference microservice per platform. A single C++ server binary plus tiny clients keeps every integration small.
- **Workflow over prompt.** Turning a raw model into useful behaviour (RAG, guardrails, tool calls, canned refusals, …) needs composition. Tryll makes the composition first-class by giving you a graph instead of a prompt template.
- **Shared, not duplicated, compute.** Models are big, KV caches are bigger. The server loads each model once and lets every agent share it; evict rules are explicit ([model management](https://docs.tryllengine.com/reference/model-management.html)).

## Edges and pitfalls

- A session is owned by its socket. If the TCP connection drops, the server destroys every agent and storage created through it. Client libraries surface this as a connection-changed event — reconnect and re-create what you need.
- A [KV cache](https://docs.tryllengine.com/reference/glossary.html#context-kv) is per node, not per agent. Two nodes in the same graph that use the same model still have separate caches; that is intentional, because they see different prompt prefixes. See [Projection and Token Budgets](https://docs.tryllengine.com/concepts/projection-and-token-budgets.html).
- The server never runs user-supplied code. Tool calls are *detection-only* — see [Tool Calling](https://docs.tryllengine.com/concepts/tool-calling.html).

## Related

- [Reference: Wire Protocol](https://docs.tryllengine.com/reference/wire-protocol.html)
- [Reference: Server Configuration](https://docs.tryllengine.com/reference/server-configuration.html)
- [Concept: Agents and Sessions](https://docs.tryllengine.com/concepts/agents-and-sessions.html)
- [Concept: Workflows, Graphs, and Nodes](https://docs.tryllengine.com/concepts/workflows-graphs-nodes.html)
- [Concept: Models and Inference Engines](https://docs.tryllengine.com/concepts/models-and-inference-engines.html)
- [How-to: Run the Tryll Server](https://docs.tryllengine.com/how-to/run-the-tryll-server.html)
