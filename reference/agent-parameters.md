# Agent Parameters

Every [agent](https://docs.tryllengine.com/reference/glossary.html#agent) in Tryll is created by a single wire frame: `CreateAgentRequest`. This page documents every field on that request, grouped by what it controls:

1. The [graph](#graph) the agent will run.
1. The [fallback model](#default-model) every [Generate node](https://docs.tryllengine.com/reference/nodes/generate.html) uses by default.
1. [Diagnostics](#diagnostics).
1. The agent's [variables](#variables) declaration.
1. [KV-cache initialization](#kv-cache-initialization).

The authoritative source for every shape below is the [wire protocol](https://docs.tryllengine.com/reference/wire-protocol.html).

Changing parameters after creation

Many node parameters can be updated after the agent is created and while it is idle. See [Change Agent Parameters at Runtime](https://docs.tryllengine.com/how-to/change-agent-parameters.html) for the full list and API examples.

## Overview

| Field                       | Type                         | Required | Default        | Description                                                                                                                                                                                                                                                                                                                                                                                |
| --------------------------- | ---------------------------- | -------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `request_id`                | uint64                       | Yes      | —              | Client-assigned correlation id. Echoed on `CreateAgentResponse` and on any `ErrorResponse` replacing it.                                                                                                                                                                                                                                                                                   |
| `graph`                     | `GraphDescription`           | Yes      | —              | The workflow graph. See [Graph](#graph).                                                                                                                                                                                                                                                                                                                                                   |
| `default_model_name`        | string                       | No       | `""`           | Fallback model name for Generate nodes that do not set their own `model_name` param. Empty = no default; every Generate node must supply its own.                                                                                                                                                                                                                                          |
| `enable_diagnostics`        | bool                         | No       | `false`        | When `true`, the server serialises per-node execution data into `TurnComplete.debug_info`.                                                                                                                                                                                                                                                                                                 |
| `maintain_dialogue_history` | bool                         | No       | `true`         | When `true` (default) the agent keeps its full dialogue history across turns. When `false` the history is cleared after each turn completes — a **stateless agent** whose turns start with only the system prompt and the new message. Useful for technical agents (classification, routing, tool-picking) that gain nothing from prior turns and should not pay to store or project them. |
| `variables`                 | `[AgentVariable]`            | No       | `[]`           | The agent's **complete** variable declaration — every variable it will ever have, with its initial value. The value's type fixes that variable's type for the agent's lifetime; post-create writes to an undeclared name are rejected. See [Variables](#variables).                                                                                                                        |
| `kv_cache_initialization`   | `AgentKvCacheInitialization` | No       | `AllocateOnly` | How the agent's language-model KV caches are prepared at creation: `AllocateOnly` (default), `Prefill`, or `DeferAllocation`. See [KV-cache initialization](#kv-cache-initialization).                                                                                                                                                                                                     |

A `CreateAgentRequest` is rejected with error [`3003`](https://docs.tryllengine.com/reference/error-codes.html#3xxx-agent) `GraphCompilationFailed` if any validation rule below fails.

______________________________________________________________________

## Graph

`GraphDescription` carries the full topology of the agent's workflow.

| Field        | Type                | Description                                                            |
| ------------ | ------------------- | ---------------------------------------------------------------------- |
| `nodes`      | `[NodeDescription]` | List of every [node](#nodedescription) in the graph. At least one.     |
| `start_node` | string              | Name of the node where execution begins. Must match a node in `nodes`. |

Wiring between nodes is encoded in each node's typed **params** object, not in a separate list. Each declared exit has a corresponding `<exit_name>_exit` field (e.g. `default_exit`, `triggered_exit`). An **empty string** routes the turn to END. A **non-empty string** must name another node in `nodes` — the server rejects graphs where an exit field names a missing node with error [`3008 InvalidExitTarget`](https://docs.tryllengine.com/reference/error-codes.html#3xxx-agent).

### `NodeDescription`

| Field         | Type               | Description                                                                                                                                                                                                                         |
| ------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`        | string             | Node instance name. Unique within the graph. Also the default `source` label on Retrieve.                                                                                                                                           |
| `params_type` | union tag          | Selects the typed params table variant (`GenerateParams`, `RegexGuardrailParams`, etc.).                                                                                                                                            |
| `params`      | typed table        | The typed params table for this node type. Contains configuration fields (model name, prompts, …) **and** exit fields (`default_exit`, `triggered_exit`, …). Each node's reference page lists all available params and exit fields. |
| `tools`       | `[ToolDefinition]` | Tool schemas. Used only by `ToolCall` nodes; ignored by all others.                                                                                                                                                                 |

### `ToolDefinition`

| Field         | Type                    | Description                                                      |
| ------------- | ----------------------- | ---------------------------------------------------------------- |
| `name`        | string                  | Tool name exposed to the model.                                  |
| `description` | string                  | Short description shown to the model as part of the tool prompt. |
| `parameters`  | `[ToolParamDefinition]` | Parameter schema for the tool.                                   |

#### `ToolParamDefinition`

| Field         | Type   | Description                                                                                                      |
| ------------- | ------ | ---------------------------------------------------------------------------------------------------------------- |
| `name`        | string | Parameter name.                                                                                                  |
| `type`        | string | Type hint as a string (e.g. `"string"`, `"integer"`). Model-facing only; the server does not validate arguments. |
| `description` | string | Short description of the parameter.                                                                              |

See [Tool Call node](https://docs.tryllengine.com/reference/nodes/tool-call.html) and the [tool-calling concept page](https://docs.tryllengine.com/concepts/tool-calling.html) for how the server uses these definitions.

______________________________________________________________________

## Default model

`default_model_name` is a fallback used by Generate nodes that omit their own `model_name` param. Leaving it empty is legal — every Generate node must then supply `model_name` itself; at least one node missing both results in a compilation failure.

Naming resolves against the [model catalog](https://docs.tryllengine.com/reference/glossary.html#model-catalog) (`models.json`). The server loads the model on demand if it is not already resident; see [model management](https://docs.tryllengine.com/reference/model-management.html#lifecycle).

______________________________________________________________________

## Diagnostics

When `enable_diagnostics = true`, every `TurnComplete` that ends successfully carries a JSON string in `debug_info` with:

- Per-node entry / exit events, in execution order.
- The [exit route](https://docs.tryllengine.com/reference/glossary.html#exit-route) taken at each branching node.
- The prompt passed to each Generate node (on servers where this is enabled).

When the server-side `include_interaction_in_diagnostics` flag (in [`server-config.json`](https://docs.tryllengine.com/reference/server-configuration.html)) is **also** set, `debug_info` additionally carries the [interaction](https://docs.tryllengine.com/reference/glossary.html#interaction)'s knowledge components, each as a flat JSON object with a `"_type"` discriminator.

The default is `false` — no diagnostics, zero overhead. Flip it on for debug sessions only; payloads grow linearly with graph size and knowledge component count.

See [How to define and handle tool calls](https://docs.tryllengine.com/how-to/define-and-handle-tool-calls.html) for a worked example that turns diagnostics on to inspect the tool-call detection path.

______________________________________________________________________

## Variables

`variables` declares the agent's typed per-agent key→value store, **complete** at creation. Each `AgentVariable` carries a name and an initial value whose active type (`int64` / `double` / `string` / `bool` / `set<string>`) fixes that variable's type for the agent's lifetime.

- Consumed by Mustache templates as `{{var.<name>}}`, by the Retrieve / ClassifyIntent filter grammar as a `{"var": "<name>"}` operand, and — when a `Generate` / `GenerateAndSpeak` node sets `substitute_agent_variables` — by `__NAME__` marker substitution in streamed output.
- Mutated over the wire with `UpdateAgentVariablesRequest`; writes to an undeclared name are rejected locally without a round-trip. An invalid declaration fails creation with [`3015 InvalidVariableValue`](https://docs.tryllengine.com/reference/error-codes.html#3xxx-agent); runtime write errors are [`3013 UnknownVariable`](https://docs.tryllengine.com/reference/error-codes.html#3xxx-agent) and [`3014 VariableTypeMismatch`](https://docs.tryllengine.com/reference/error-codes.html#3xxx-agent).

See [Concept: Agent Variables](https://docs.tryllengine.com/concepts/agent-variables.html) and the how-tos [Drive prompts and retrieval from game state](https://docs.tryllengine.com/how-to/drive-prompts-and-retrieval-from-game-state.html) and [Substitute agent variables in LLM output](https://docs.tryllengine.com/how-to/substitute-agent-variables-in-output.html).

______________________________________________________________________

## KV-cache initialization

`kv_cache_initialization` chooses how the agent's eligible language-model contexts (LLM nodes only — embeddings and Sherpa STT/TTS are unaffected) are prepared when the agent is created:

| Value                    | Behavior                                                                                                                                                                                                                                          |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `AllocateOnly` (default) | Allocate each context's KV cache but do not decode anything yet.                                                                                                                                                                                  |
| `Prefill`                | Allocate **and** decode each context's reusable prefix up front, so the first turn skips that cost. Prefill failure surfaces as [`4004 KvCacheOperationFailed`](https://docs.tryllengine.com/reference/error-codes.html#4xxx-inference-workflow). |
| `DeferAllocation`        | Defer even allocation until the first send.                                                                                                                                                                                                       |

Idle agents also support the aggregate `PrefillAgentKvCache`, `EvictAgentKvCache`, and `GetAgentKvCacheStatus` operations; eligible contexts are restored automatically before every send. See [Manage an agent's KV cache](https://docs.tryllengine.com/how-to/manage-agent-kv-cache.html).

______________________________________________________________________

## Minimum working example

```
# Minimal agent with no Retrieve nodes
from tryll_client.graph import GraphDescription, GenerateParams

graph = (
    GraphDescription()
    .add_node("answer", GenerateParams())
    .set_start_node("answer")
    .set_default_model_name("My Local Model")
)

agent = client.create_agent(graph)
```

```
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

GenerateParamsT gp;
// default_exit defaults to "" (END)

GraphDescription graph;
graph.AddGenerate("answer", std::move(gp))
     .SetStartNode("answer")
     .SetDefaultModelName("My Local Model");

auto agent = client.CreateAgent(graph);
```

```
UTryllGenerateParams* P = UTryllNodeParamsFactory::MakeGenerateParams(this);
// DefaultExit defaults to TEXT("") (END)

FTryllGraphDescription Graph = FTryllGraphBuilder()
    .AddNode(TEXT("answer"), P)
    .SetStartNode(TEXT("answer"))
    .SetDefaultModelName(TEXT("My Local Model"))
    .Build();
```

Or author the equivalent inside a `UTryllWorkflowAsset` and assign it to the `UTryllAgentComponent`.

For an agent with Retrieve + Mustache template, see [How to create a simple RAG assistant](https://docs.tryllengine.com/how-to/create-simple-rag-assistant.html) and [How to use Mustache templates](https://docs.tryllengine.com/how-to/use-mustache-templates.html).

## Client bindings

- **C++:** `Tryll::Client::TryllClient::CreateAgent` + `Tryll::Client::GraphDescription` — `TryllClient.h`, `GraphDescription.h`
- **Python:** `tryll_client.TryllClient.create_agent` + `tryll_client.GraphDescription` — `client.py`, `graph.py`
- **Unreal:** `UTryllAgentComponent::CreateAgent` / `FTryllGraphDescription` / `UTryllWorkflowAsset` — `TryllGraphDescription.h`

## Related

- [Node catalog](https://docs.tryllengine.com/reference/nodes/index.html)
- [How to build a chat agent with a graph](https://docs.tryllengine.com/how-to/build-chat-agent-with-graph.html)
- [How to use Mustache templates for prompt projection](https://docs.tryllengine.com/how-to/use-mustache-templates.html)
- [Concept: Workflows, graphs, and nodes](https://docs.tryllengine.com/concepts/workflows-graphs-nodes.html)
- [Glossary](https://docs.tryllengine.com/reference/glossary.html)
