# Agents and Sessions

Everything a client does with Tryll happens inside a *session*, and every conversation inside that session is an *agent*. Understanding how these two objects come into being, what they own, and what happens when they go away is the single most useful piece of mechanical knowledge for integrating the server.

## Session: one TCP connection, one world

In protocol v4 the TCP **connection** and the logical **session** are distinct. The connection is created when the server accepts your socket; the [session](https://docs.tryllengine.com/reference/glossary.html#session) is created only when you send `CreateSession`. From the client's side, the steps are:

1. `Client::connect(host, port)` opens the socket.
1. The server sends an unsolicited `ConnectionReady` frame — a connection-level hello (it carries the protocol version and codegen fingerprint, but **no** `session_id`).
1. You send `CreateSessionRequest` to create the session and pick its inference engines (for example `LlamaCpp` or `Mock`). This is **mandatory and one-shot**: it must precede every other request, and the engines/flags are **fixed for the life of the session** (to change them, reconnect). The server replies with `CreateSessionResponse`, which carries the server-allocated `session_id`.
1. Only now can you create per-session helpers ([string storages](https://docs.tryllengine.com/reference/string-storage.html), [embedded string storages](https://docs.tryllengine.com/reference/embedded-string-storage.html)), manage models, and create agents. Any of these before `CreateSession` is rejected with `SessionNotReady`; a second `CreateSession` with `SessionAlreadyExists`.

Everything in steps 3 and 4 belongs to the session. When the socket closes, the server:

- cancels every active agent turn,
- destroys every agent in the session,
- drops every string storage and embedded storage the session owned,
- frees any on-demand language models that no other session still references (see [Model Management](https://docs.tryllengine.com/reference/model-management.html)).

A process can hold many sessions; a session holds many agents; and a single server process is shared across all of them. If the socket drops, the client re-connects and re-creates what it needs — there is no server-side persistence for session state.

### The implicit session state machine

Sessions do not have an explicit state enum, but they behave like a tiny state machine:

```
stateDiagram-v2
    direction LR
    [*]            --> Connected:        TCP accept + ConnectionReady
    Connected      --> Connected:        any non-CreateSession request → SessionNotReady
    Connected      --> SessionActive:    CreateSessionRequest (mandatory, one-shot)
    SessionActive  --> SessionActive:    second CreateSession → SessionAlreadyExists
    SessionActive  --> AgentActive:      CreateAgentRequest
    AgentActive    --> AgentActive:      SendMessage / DownloadModel / Load / …
    AgentActive    --> SessionActive:    last agent destroyed
    Connected      --> Closing:          socket EOF / server shutdown
    SessionActive  --> Closing:          socket EOF / server shutdown
    AgentActive    --> Closing:          socket EOF / server shutdown
    Closing        --> [*]:              reader + writer exit
```

`CreateSession` is valid exactly once, and only on a bare connection. Model management, storage, voice, and agent requests are valid only **after** `CreateSession` — you can warm up a model before creating any agents, but not before creating the session.

## Agent: one conversation, one graph

An [agent](https://docs.tryllengine.com/reference/glossary.html#agent) is a single ongoing conversation. You create one by sending `CreateAgentRequest` with:

- a [workflow graph](https://docs.tryllengine.com/concepts/workflows-graphs-nodes.html) (required),
- a default model name for nodes that do not override it,
- optional `template` and `placement` params on `Generate` nodes (see [Use Mustache Templates](https://docs.tryllengine.com/how-to/use-mustache-templates.html)) if the graph contains any `Retrieve` or `Instruction` node,
- an optional [`variables`](https://docs.tryllengine.com/concepts/agent-variables.html) declaration (the agent's complete typed key→value store) and a `kv_cache_initialization` choice (`AllocateOnly` / `Prefill` / `DeferAllocation`) — see [Agent Parameters](https://docs.tryllengine.com/reference/agent-parameters.html) and [Manage an Agent's KV Cache](https://docs.tryllengine.com/how-to/manage-agent-kv-cache.html),
- optional per-agent flags like `enable_diagnostics`, `maintain_dialogue_history`, and the `max_steps_per_turn` budget.

The server responds with `CreateAgentResponse` once the graph compiles and the default model resolves — after that you can send messages. Models referenced by the graph must already be on disk; `CreateAgent` fails fast with `GraphCompilationFailed` if one is missing, rather than downloading it on the spot. Acquire models beforehand via the editor Model Manager or `DownloadModelRequest`.

A session can own as many agents as you want. Each has its own dialog (the growing list of [interactions](https://docs.tryllengine.com/reference/glossary.html#interaction)), its own per-node KV caches, and its own sampling parameters. Turns on a single session run one at a time — agent A's turn finishes before agent B's turn starts. The dialog is not a save file: to persist it across sessions, export a snapshot into **your** game save. See [Save and load agent state](https://docs.tryllengine.com/how-to/save-and-load-agent-state.html).

### The implicit agent state machine

```
stateDiagram-v2
    direction LR
    [*]     --> Idle:    CreateAgent compiled the graph
    Idle    --> Running: SendMessage → turn starts
    Running --> Running: nodes execute; inference runs on the shared model
    Running --> Paused:  Pause node / pausing ToolCall disposition
    Paused  --> Running: Resume (continue exit route, or jump to a node)
    Running --> Idle:    graph reaches END → TurnComplete(Success)
    Running --> Idle:    Cancel / teardown → TurnComplete(Cancelled)
    Running --> Idle:    max_steps exceeded / error → TurnComplete(Error)
    Paused  --> Idle:    Cancel → TurnComplete(Cancelled)
    Idle    --> [*]:     DestroyAgent or session close
    Running --> [*]:     session close (turn is cancelled first)
```

Four facts about this machine matter in practice:

1. **Turns are atomic per agent.** A second `SendMessage` while the previous turn is running is rejected with [error 3004 `AgentBusy`](https://docs.tryllengine.com/reference/error-codes.html#3xxx-agent). The in-flight turn is *not* pre-empted; it runs to completion.
1. **A turn can pause between nodes.** A [`Pause`](https://docs.tryllengine.com/reference/nodes/pause.html) node, or a [`ToolCall`](https://docs.tryllengine.com/reference/nodes/tool-call.html) with a pausing `disposition` (`Pause`, `PauseAndAcknowledge`, `AwaitResult`), suspends the turn mid-flight. While **Paused** the turn is still open (`SendMessage` still returns `AgentBusy`), but [`ChangeAgentParam`](https://docs.tryllengine.com/how-to/change-agent-parameters.html) — normally forbidden while a turn runs — is allowed. `Resume` continues it; there is no pause timeout. See [Pause and Resume a Turn](https://docs.tryllengine.com/how-to/pause-and-resume-turns.html).
1. **Cancellation is cooperative.** A [`Cancel`](https://docs.tryllengine.com/how-to/cancel-a-turn.html), destroying the agent, or closing the session signals the turn to stop; it unwinds at the next convenient point and produces `TurnComplete(Cancelled)`.
1. **Step budget, not time budget.** The server aborts a turn if the graph walks more than `max_steps_per_turn` nodes (default 64). This is the escape valve for routing loops, not a generation timeout.

## What an agent owns

```
flowchart TB
    A[Agent] --> D[Dialog<br>list of Interactions]
    A --> G[Graph<br>frozen at CreateAgent]
    A --> N1[Node 1<br>KV cache / params]
    A --> N2[Node 2<br>KV cache / params]
    A --> N3[Node N<br>...]
    G --> ExitFields[exit fields on params]
    G --> Start[start id]
```

- The **dialog** is the source of truth for the conversation. Every turn appends one interaction: the user's message, the model's reply, and anything the graph attached along the way (retrieved knowledge, recorded tool calls).
- The **graph** is frozen for the life of the agent. You cannot swap nodes or exit wiring after `CreateAgent` — destroy and re-create the agent if you need a different shape.
- **Per-node state** — KV caches, sampling defaults, and any node-local caches — belongs to the agent, not the session. Two agents using the same model still have independent KV state.

## Lifecycle in one picture

```
sequenceDiagram
    participant C as Client
    participant S as Session
    participant A as Agent

    C->>S: connect
    S-->>C: ConnectionReady
    C->>S: CreateSessionRequest
    S-->>C: CreateSessionResponse
    C->>S: CreateAgentRequest(graph, model, …)
    S->>A: compile graph, resolve model
    S-->>C: CreateAgentResponse(agent_id)
    loop one per user turn
        C->>S: SendMessageRequest(agent_id, text)
        S->>A: dispatch
        A-->>S: AnswerText (stream)
        S-->>C: AnswerText
        A-->>S: TurnComplete(status)
        S-->>C: TurnComplete
    end
    C->>S: DestroyAgentRequest(agent_id)
    S->>A: cancel + destroy
    S-->>C: Ack
    C->>S: disconnect
```

## Edges and pitfalls

- **Agent ids are session-scoped.** An `agent_id` is only meaningful inside the session that created it. If you reconnect, all previous agent ids become invalid.
- **`CreateSession` is one-shot; engines are fixed per session.** You cannot change the inference engine after the session is created — a second `CreateSession` is rejected with `SessionAlreadyExists`. To run with a different engine, open a new connection and create a new session.
- **Shared string storages need the storage alive at `CreateAgent` time.** Destroying a [string storage](https://docs.tryllengine.com/reference/string-storage.html) after creating an agent that references it is safe — the agent keeps using the data until the agent itself is destroyed. But the reverse is not: you cannot reference a storage that does not yet exist. See [Lifetime and Ownership](https://docs.tryllengine.com/concepts/lifetime-and-ownership.html) for the full picture of who holds what reference and when each goes away.
- **Multiple agents on one session run serially.** The server multiplexes their turns fairly, but on a given session only one turn runs at a time. Open more sessions if you need parallelism.

## Related

- [Reference: Agent Parameters](https://docs.tryllengine.com/reference/agent-parameters.html)
- [Reference: Wire Protocol](https://docs.tryllengine.com/reference/wire-protocol.html)
- [Reference: Error Codes](https://docs.tryllengine.com/reference/error-codes.html)
- [Concept: Architecture at a Glance](https://docs.tryllengine.com/concepts/architecture-at-a-glance.html)
- [Concept: Lifetime and Ownership](https://docs.tryllengine.com/concepts/lifetime-and-ownership.html)
- [Concept: Workflows, Graphs, and Nodes](https://docs.tryllengine.com/concepts/workflows-graphs-nodes.html)
- [How-to: Connect and manage a session](https://docs.tryllengine.com/how-to/connect-and-manage-session.html)
- [How-to: Build a chat agent with a graph](https://docs.tryllengine.com/how-to/build-chat-agent-with-graph.html)
