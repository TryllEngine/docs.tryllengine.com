# Agents and Sessions

Everything a client does with Tryll happens inside a *session*, and every conversation inside that session is an *agent*. Understanding how these two objects come into being, what they own, and what happens when they go away is the single most useful piece of mechanical knowledge for integrating the server.

## Session: one per connection, one world

The TCP **connection** and the logical **session** are distinct. The connection is created when the server accepts your socket; the [session](https://docs.tryllengine.com/v0.10/reference/glossary.html#session) is created only when you send `CreateSession`. From the client's side, the steps are:

1. `Connect(host, port)` opens the socket.
1. The server sends an unsolicited `ConnectionReady` frame — a connection-level hello (it carries the protocol version and codegen fingerprint, but **no** `session_id`). The client compares `protocol_version` (currently 29) with its own and fails the connection locally with `5004 ProtocolVersionMismatch` if they differ; the server never sends 5004.
1. You send `CreateSessionRequest` to create the session. This is **mandatory and one-shot**: it must precede every other request. It sets session-wide options such as the game name and locale (the locale can be changed later with `SetSessionLocale`). It does not choose an inference engine — each catalog model names its own engine. Its `model_config` may carry an optional list of `(role, engine)` hints that only warm those engines early. The server replies with `CreateSessionResponse`, which carries the server-allocated `session_id`.
1. Only now can you create per-session helpers ([string storages](https://docs.tryllengine.com/v0.10/reference/string-storage.html), [embedded string storages](https://docs.tryllengine.com/v0.10/reference/embedded-string-storage.html)), manage models, and create agents. Any of these before `CreateSession` is rejected with `SessionNotReady`; a second `CreateSession` with `SessionAlreadyExists`.

Everything in steps 3 and 4 belongs to the session. When the socket closes, the server:

- cancels every active agent turn,
- destroys every agent in the session,
- drops every string storage and embedded storage the session owned.

Models are not unloaded at that moment. An on-demand model that no other session holds stays cached until the next session's pre-load sweep drops it; pinned models stay loaded (see [Model Management](https://docs.tryllengine.com/v0.10/reference/model-management.html)).

A process can hold many sessions; a session holds many agents; and a single server process is shared across all of them. If the socket drops, the client re-connects and re-creates what it needs — there is no server-side persistence for session state.

### The implicit session state machine

Sessions do not have an explicit state enum, but they behave like a tiny state machine:

```
stateDiagram-v2
    direction TB
    [*] --> Connected: TCP accept
    Connected --> Connected: request before CreateSession
    Connected --> SessionActive: CreateSession
    SessionActive --> SessionActive: second CreateSession
    SessionActive --> AgentActive: CreateAgent
    AgentActive --> AgentActive: SendMessage and other requests
    AgentActive --> SessionActive: last agent destroyed
    Connected --> Closing: socket closed
    SessionActive --> Closing: socket closed
    AgentActive --> Closing: socket closed
    Closing --> [*]: reader and writer exit
```

`CreateSession` is valid exactly once, and only on a bare connection. Model management, storage, voice, and agent requests are valid only **after** `CreateSession` — you can warm up a model before creating any agents, but not before creating the session.

## Agent: one conversation, one graph

An [agent](https://docs.tryllengine.com/v0.10/reference/glossary.html#agent) is a single ongoing conversation. You create one by sending `CreateAgentRequest` with:

- a [workflow graph](https://docs.tryllengine.com/v0.10/concepts/workflows-graphs-nodes.html) (required),
- a default model name for nodes that do not override it,
- optional `template` and `placement` params on `Generate` nodes (see [Use Mustache Templates](https://docs.tryllengine.com/v0.10/how-to/use-mustache-templates.html)) if the graph contains any `Retrieve` or `Instruction` node,
- an optional [`variables`](https://docs.tryllengine.com/v0.10/concepts/agent-variables.html) declaration (the agent's complete typed key→value store) and a `kv_cache_initialization` choice (`AllocateOnly` / `Prefill` / `DeferAllocation`) — see [Agent Parameters](https://docs.tryllengine.com/v0.10/reference/agent-parameters.html) and [Manage an Agent's KV Cache](https://docs.tryllengine.com/v0.10/how-to/manage-agent-kv-cache.html),
- a [voice profile](https://docs.tryllengine.com/v0.10/reference/voice-profile.html) (`voice`), required when the graph has a `Speak` or `GenerateAndSpeak` node — the agent's one voice, shared by every speaking node,
- optional per-agent flags like `enable_diagnostics` and `maintain_dialogue_history`.

The server responds with `CreateAgentResponse` once the graph compiles and the default model resolves — after that you can send messages. Models referenced by the graph must already be on disk; `CreateAgent` fails fast with `GraphCompilationFailed` if one is missing, rather than downloading it on the spot. Acquire models beforehand via the editor Model Manager or `DownloadModelRequest`.

A session can own as many agents as you want. Each has its own dialog (the growing list of [interactions](https://docs.tryllengine.com/v0.10/reference/glossary.html#interaction)), its own per-node KV caches, and its own sampling parameters. Each agent runs one turn at a time: a second `SendMessage` to a busy agent gets `AgentBusy`. Turns on different agents of the same session may overlap; agents that share a loaded model still queue on that model's scheduler in FIFO order. The dialog is not a save file: to persist it across sessions, export a snapshot into **your** game save. See [Save and load agent state](https://docs.tryllengine.com/v0.10/how-to/save-and-load-agent-state.html).

### The implicit agent state machine

```
stateDiagram-v2
    direction TB
    [*] --> Idle: CreateAgent
    Idle --> Running: SendMessage
    Running --> Running: execute
    Running --> Paused: pause
    Paused --> Running: Resume
    Running --> Idle: success
    Running --> Idle: cancelled
    Running --> Idle: error
    Paused --> Idle: cancel
    Paused --> Idle: timed out
    Idle --> [*]: destroy
    Running --> [*]: close
```

Four facts about this machine matter in practice:

1. **Turns are atomic per agent.** A second `SendMessage` while the previous turn is running is rejected with [error 3004 `AgentBusy`](https://docs.tryllengine.com/v0.10/reference/error-codes.html#3xxx-agent). The in-flight turn is *not* pre-empted; it runs to completion.
1. **A turn can pause between nodes.** A [`Pause`](https://docs.tryllengine.com/v0.10/reference/nodes/pause.html) node, or a [`ToolCall`](https://docs.tryllengine.com/v0.10/reference/nodes/tool-call.html) with a pausing `disposition` (`Pause`, `PauseAndAcknowledge`, `AwaitResult`), suspends the turn mid-flight. While **Paused** the turn is still open (`SendMessage` still returns `AgentBusy`), but [`ChangeAgentParam`](https://docs.tryllengine.com/v0.10/how-to/change-agent-parameters.html) — normally forbidden while a turn runs — is allowed. `Resume` continues it. A pause can also time out: `Pause.pause_timeout_ms` of `0` waits forever, and with a timeout the turn follows `timeout_exit`, or ends with `TurnComplete(PauseTimedOut)` when that exit is empty. A `ToolCall` with `pause_timeout_ms` `0` inherits the server's `workflow.tool_call_pause_timeout_ms` (`0`, wait forever, by default and in the released server; `2000` in a source checkout's `data/server-config.json`) and has no timeout exit. See [Pause and Resume a Turn](https://docs.tryllengine.com/v0.10/how-to/pause-and-resume-turns.html).
1. **Cancellation is cooperative.** A [`Cancel`](https://docs.tryllengine.com/v0.10/how-to/cancel-a-turn.html), destroying the agent, or closing the session signals the turn to stop; it unwinds at the next convenient point and produces `TurnComplete(Cancelled)`.
1. **Step budget, not time budget.** Every agent stops a turn after 64 node steps. The budget is fixed, not a `CreateAgent` setting. This is the escape valve for routing loops, not a generation timeout.

## What an agent owns

```
flowchart TB
    A[Agent] --> owned
    subgraph owned [owns]
        direction TB
        D[Dialog]
        G[Graph]
        V[Voice]
        N[Nodes and KV caches]
    end
    G --> E[exit fields and start id]
```

- The **dialog** is the source of truth for the conversation. Every turn appends one interaction: the user's message, the model's reply, and anything the graph attached along the way (retrieved knowledge, recorded tool calls).
- The **graph** is frozen for the life of the agent. You cannot swap nodes or exit wiring after `CreateAgent` — destroy and re-create the agent if you need a different shape.
- **Per-node state** — KV caches, sampling defaults, and any node-local caches — belongs to the agent, not the session. Two agents using the same model still have independent KV state.
- The **voice** belongs to the agent, not to its graph. Every speaking node uses it, so one graph can serve characters with different voices. Replace it with `SetAgentVoice` while the agent is idle, and switch its [delivery mode](https://docs.tryllengine.com/v0.10/reference/voice-profile.html#delivery-modes) with `SetAgentDeliveryMode` at any time. Agents that use the same TTS model share the loaded model.

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
- **`CreateSession` is one-shot.** A second `CreateSession` is rejected with `SessionAlreadyExists`; to start over with different settings, open a new connection and create a new session. Engines are not a session setting: each model runs on the engine its catalog entry names, whatever engine hints the session sent.
- **Shared string storages need the storage alive at `CreateAgent` time.** Destroying a [string storage](https://docs.tryllengine.com/v0.10/reference/string-storage.html) after creating an agent that references it is safe — the agent keeps using the data until the agent itself is destroyed. But the reverse is not: you cannot reference a storage that does not yet exist. See [Lifetime and Ownership](https://docs.tryllengine.com/v0.10/concepts/lifetime-and-ownership.html) for the full picture of who holds what reference and when each goes away.
- **`AgentBusy` is per agent, not per session.** A second `SendMessage` to an agent that is mid-turn is rejected with `3004`. Other agents on the same session can start their turns at the same time. Inference on one loaded model stays FIFO on that model's scheduler, so two agents on the same model take turns at the GPU.

## Related

- [Reference: Agent Parameters](https://docs.tryllengine.com/v0.10/reference/agent-parameters.html)
- [Reference: Wire Protocol](https://docs.tryllengine.com/v0.10/reference/wire-protocol.html)
- [Reference: Error Codes](https://docs.tryllengine.com/v0.10/reference/error-codes.html)
- [Concept: Architecture at a Glance](https://docs.tryllengine.com/v0.10/concepts/architecture-at-a-glance.html)
- [Concept: Lifetime and Ownership](https://docs.tryllengine.com/v0.10/concepts/lifetime-and-ownership.html)
- [Concept: Workflows, Graphs, and Nodes](https://docs.tryllengine.com/v0.10/concepts/workflows-graphs-nodes.html)
- [How-to: Connect and manage a session](https://docs.tryllengine.com/v0.10/how-to/connect-and-manage-session.html)
- [How-to: Build a chat agent with a graph](https://docs.tryllengine.com/v0.10/how-to/build-chat-agent-with-graph.html)
