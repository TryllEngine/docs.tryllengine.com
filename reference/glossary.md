# Glossary

Canonical definitions for every term used across the Tryll documentation. Every page links the **first mention** of a term below to its anchor on this page. Subsequent mentions render as plain text.

Terms are grouped by topic so related concepts sit next to each other.

______________________________________________________________________

## Sessions, agents, and turns

### Agent

A server-side object that owns one conversation: its [dialog](#dialog), its compiled [workflow](#workflow), its retained knowledge, and a handle to the [models](#language-model) it needs. Each agent belongs to exactly one [session](#session) and is referenced by a numeric `agent_id`. Created via `CreateAgentRequest`, retired via `DestroyAgentRequest`.

### Session

A single configured connection between a client and the Tryll server. The client opens a TCP socket, sends a `CreateSessionRequest`, and receives a `session_id`. All subsequent requests from that socket are scoped to the session; closing the socket ends the session and destroys every [agent](#agent) it owns.

### Turn

One request–response cycle initiated by a user message. A turn begins when the client sends `SendMessageRequest` and ends when the server emits `TurnComplete`. Between those two frames the client may receive zero or more `AnswerText` chunks (see [streaming](#streaming)). Only one turn is in flight per agent at a time.

### Dialog

The ordered list of interactions belonging to an [agent](#agent) — the conversation as the server sees it. Each [turn](#turn) appends exactly one [interaction](#interaction). Used by the [projection](#projection) stage to build the next [prompt](#prompt).

### Interaction

A single user-and-assistant exchange within a [dialog](#dialog). Holds the user message, the assistant response, any retrieved [knowledge components](#knowledge-component), and (when diagnostics are on) per-node debug information.

______________________________________________________________________

## Graphs, nodes, and workflows

### Workflow

The overall per-turn behaviour of an [agent](#agent) — what happens between receiving the user message and emitting `TurnComplete`. A workflow is defined by a [graph](#graph) of [nodes](#node) and is compiled from a `GraphDescription` when the agent is created.

### Graph

A declarative description of a [workflow](#workflow): a list of [nodes](#node), where each node's typed params carry both its configuration and its [exit](#exit-route) wiring, plus a start node name. Built client-side with the graph builder APIs, validated server-side at agent creation. Compilation failures surface as error `3003` (see [error codes](https://docs.tryllengine.com/reference/error-codes.html)).

### Node

A single unit of computation within a [graph](#graph). Each node has a typed parameter struct containing both its configuration and one or more named [exit](#exit-route) fields, and well-defined side effects on the [dialog](#dialog). See [Workflow Nodes](https://docs.tryllengine.com/reference/nodes/index.html) for the full, current catalog of built-in node types.

### Exit route

A named output of a [node](#node). Each exit is encoded as a typed string field on the node's params object using the naming convention `<exit_name>_exit` (e.g. `triggered_exit`, `not_triggered_exit`). An empty string routes the turn to end-of-turn; any non-empty value names the next node to execute. For example, `RegexGuardrailNode` has `triggered` and `not_triggered` exits; `ToolCallNode` has `tool_called` and `no_tool_called`.

### Projection

The stage that turns the [dialog](#dialog) plus retained state into a [prompt](#prompt) for a [language model](#language-model). For `Generate` nodes, projection also renders the node's [Mustache template](#mustache-template) against the current turn's [instruction slots](#instruction-slot) and [knowledge components](#knowledge-component), then splices the result into the prompt at the configured placement. A token budget with hysteresis ensures the prompt fits inside the model's context window. See the [projection concept page](https://docs.tryllengine.com/concepts/projection-and-token-budgets.html).

### Mustache template

A [Mustache](https://mustache.github.io/) source string attached to a `Generate` node via the `template` param. Before each model call, [projection](#projection) renders it against a context that includes `{{user_message}}`, the `{{#instructions}}` list from [instruction slots](#instruction-slot), and the `{{#knowledge}}` list from [knowledge components](#knowledge-component). The rendered text is spliced into the [prompt](#prompt) at the position specified by the `placement` param. See the [Use Mustache Templates how-to](https://docs.tryllengine.com/how-to/use-mustache-templates.html).

### Classify Intent node

A [node](#node) type that maps a user message to a discrete intent label by top-1 vector similarity search over a small, labelled [embedded string storage](#embedded-string-storage). On a match it attaches an intention component to the current [interaction](#interaction) for a downstream [Intent to Instruction node](#intent-to-instruction-node) to consume; it generates no text and renders nothing into the [prompt](#prompt). See the [Classify Intent reference](https://docs.tryllengine.com/reference/nodes/classify-intent.html).

### Intent to Instruction node

A [node](#node) type that looks up the intent label written by a preceding [Classify Intent node](#classify-intent-node) in a `Map`-kind [string storage](#string-storage) and attaches the matching value as an [instruction slot](#instruction-slot) to the current [interaction](#interaction). The downstream [Generate node](https://docs.tryllengine.com/reference/nodes/generate.html) then renders it via its [Mustache template](#mustache-template). It always takes the `default` exit. See the [Intent to Instruction reference](https://docs.tryllengine.com/reference/nodes/intent-to-instruction.html).

______________________________________________________________________

## Knowledge, retrieval, and storage

### RAG

Retrieval-Augmented Generation. An approach that injects topically relevant text snippets into the [prompt](#prompt) so the [language model](#language-model) can ground its answer on content the model itself does not "know". In Tryll, RAG is implemented by a [Retrieve node](#retrieve-node) in front of a [Generate node](https://docs.tryllengine.com/reference/nodes/generate.html).

### Retrieval

The act of selecting snippets from a knowledge source that are most similar (by cosine distance) to the current query [embedding](#embedding). Produces [knowledge components](#knowledge-component) that the workflow may or may not present to the model.

### Retrieve node

A [node](#node) type that runs a vector similarity search over an [embedded string storage](#embedded-string-storage) and attaches the top-`k` matches to the current [interaction](#interaction) as [knowledge components](#knowledge-component). See the [Retrieve reference](https://docs.tryllengine.com/reference/nodes/retrieve.html).

### Knowledge base

Synonym for "vector index used by a [Retrieve node](#retrieve-node)". In Tryll, a knowledge base is realised as an [embedded string storage](#embedded-string-storage).

### Instruction slot

A named instruction string attached to the current [interaction](#interaction) as a `kind=Instruction` [slot](https://docs.tryllengine.com/concepts/slots-and-inter-node-value-passing.html) by an [Instruction node](https://docs.tryllengine.com/reference/nodes/instruction.html) (or [Intent to Instruction](https://docs.tryllengine.com/reference/nodes/intent-to-instruction.html)). [Projection](#projection) exposes instruction slots to the downstream `Generate` node's [Mustache template](#mustache-template) as: `{{slot.<name>}}` (direct lookup) and entries in the `{{#instructions}}` list (each with `{{name}}` and `{{text}}`).

### Knowledge component

A single piece of retrieved or attached text plus metadata (source, score, chunk id). Multiple components may attach to one [interaction](#interaction). [Projection](#projection) exposes them to the `Generate` node's [Mustache template](#mustache-template) as the `{{#knowledge}}` list and `{{#knowledge_<source>}}` direct-lookup sections.

### Chunk

A bounded slice of source text that has been offline-embedded and stored in an [embedded string storage](#embedded-string-storage). Chunking happens at index build time, not at query time; each [retrieval](#retrieval) result is one chunk.

### Embedding

A dense floating-point vector that represents the semantic content of a piece of text. Produced by an [embedding model](#embedding-model); compared via cosine distance inside an [HNSW](#hnsw) index.

### Embedding model

A [model](#language-model) whose output is an [embedding](#embedding) rather than generated text. Declared in the [model catalog](#model-catalog) with `"model_type": "embedding"`. Referenced by name from `RetrieveNode` parameters or from an [embedded string storage](#embedded-string-storage) config file.

### HNSW

Hierarchical Navigable Small World — the approximate-nearest-neighbour index structure used by Tryll's [embedded string storage](#embedded-string-storage). Optimised for fast cosine-distance search over large vector sets.

### String storage

A named, session-owned container of plain strings. Created via `CreateStringStorageRequest` (inline array or file path). Consumed by [canned-response](https://docs.tryllengine.com/reference/nodes/canned-response.html) and [regex-guardrail](https://docs.tryllengine.com/reference/nodes/regex-guardrail.html) nodes, which pick or match one string per invocation. Not embedded, not indexed. See the [string storage reference](https://docs.tryllengine.com/reference/string-storage.html).

### Embedded string storage

A session-owned vector index built from strings plus a precomputed [HNSW](#hnsw) file. Two construction paths: a directory containing a `*.kb.json` config plus a `*.usearch` file (Path A), or an inline array of strings that the server embeds on demand (Path B). Consumed by [Retrieve nodes](#retrieve-node). See the [embedded string storage reference](https://docs.tryllengine.com/reference/embedded-string-storage.html).

### Canned response

A fixed reply selected from a [string storage](#string-storage). The [canned-response node](https://docs.tryllengine.com/reference/nodes/canned-response.html) emits one string from its backing storage (random, first-match, or indexed pick) and terminates the turn without invoking a [language model](#language-model). Useful for scripted answers and fallback paths.

### Guardrail

A pattern-matching check that gates workflow flow. The [regex-guardrail node](https://docs.tryllengine.com/reference/nodes/regex-guardrail.html) runs an input slot (default `user_message`) against a [string storage](#string-storage) of patterns and routes the turn to `triggered` or `not_triggered` accordingly. Common use: short-circuit unsafe or off-topic inputs to a [canned response](#canned-response).

______________________________________________________________________

## Models and inference

### Language model

A model that takes a text [prompt](#prompt) and emits generated text token-by-token. In the Tryll catalog, language models omit the `model_type` field entirely — language is the implicit default. See the [models concept page](https://docs.tryllengine.com/concepts/models-and-inference-engines.html).

### SLM

Small Language Model — a [language model](#language-model) small enough to run on consumer hardware (CPU, integrated GPU, or a single consumer GPU). Tryll is designed around SLMs: on-device inference, no cloud round-trip, and aggressive KV-cache management.

### Inference engine

The backend runtime that executes a [model](#language-model). Chosen per-session on `CreateSessionRequest` as a value of the `InferenceEngine` enum. `LlamaCpp` is shipped today; `OnnxGenAI`, `WindowsML`, `OpenVino`, and `TensorRtLlm` are enum slots reserved for future engines. Each [model variant](#model-variant) targets exactly one engine.

### GGUF

The binary file format used by `llama.cpp` for quantised [language models](#language-model). Tryll downloads GGUF files from HuggingFace when the active [model variant](#model-variant) targets the `llama.cpp` [inference engine](#inference-engine).

### Model catalog

The server's `models.json` file — the authoritative list of models Tryll can download, load, and run. Each catalog entry declares a name, a purpose (`language` or `embedding`), default sampling parameters, and one or more [variants](#model-variant). See the [model management reference](https://docs.tryllengine.com/reference/model-management.html).

### Model variant

One concrete packaging of a catalogued model for one [inference engine](#inference-engine). A model named `Qwen2.5-0.5B-Instruct` may ship variants for `llama.cpp` (GGUF) and for `onnxruntime-genai` (ONNX). The active variant is determined by the server's configured default engine or by explicit selection at `LoadModelRequest` time.

### Variant

Short form of [model variant](#model-variant).

### Pinned retention

A model-loading policy that keeps a language / embedding / STT / TTS model resident in memory until an explicit `UnloadModelRequest` (and no live users remain). Suitable for the hot model(s) on a given machine. Opposite of [OnDemand retention](#ondemand-retention). VAD catalog entries are not pin/unloadable.

### OnDemand retention

A model-loading policy that loads a language / embedding / STT / TTS model when an [agent](#agent) (or voice input / TTS / embedding use) needs it and unloads it when unused — after `DestroyAgent`, or when the next validated model load sweeps idle OnDemand entries across all four caches. Trades memory for latency. Opposite of [Pinned retention](#pinned-retention).

### Context (KV)

The KV cache — the per-model tensor of cached attention keys and values for every [token](#token) the model has processed this turn. Tryll reuses as much KV state as possible across [turns](#turn) when the [prompt](#prompt) prefix is stable; this is what makes conversational inference fast.

### TTS

Text-to-speech — the synthesis of spoken audio from generated text. The [Generate and Speak node](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html) segments generated tokens into sentences and streams TTS audio frames alongside the text deltas, using a TTS [model](#language-model) named in the [model catalog](#model-catalog) via its `tts_model_name` param.

______________________________________________________________________

## Prompts, tokens, and streaming

### Prompt

The concatenated input passed to the [language model](#language-model) at [generation](https://docs.tryllengine.com/reference/nodes/generate.html) time: system preamble, projected [dialog](#dialog), attached [knowledge](#knowledge-component), tool descriptions, and the current user message. Produced by [projection](#projection).

### Token

The atomic unit the [language model](#language-model) reads and emits. Each model has its own tokenizer; token counts for the same text differ between models. Token budgets in [projection](#projection) are measured in tokens of the active model.

### Token budget

A configurable per-agent cap on how many [tokens](#token) the [projected](#projection) [prompt](#prompt) may use, reserving headroom for the generated response. Exceeding the budget triggers dialog truncation with hysteresis.

### Streaming

The mode in which the server emits generated text in chunks rather than waiting for the full response. Each chunk arrives as an `AnswerText` frame; the final chunk carries the `is_final = true` flag and is immediately followed by `TurnComplete`. See [how to stream answers to a UI](https://docs.tryllengine.com/how-to/stream-answers-to-ui.html).

______________________________________________________________________

## Tool calling

### Tool call

A structured instruction emitted by a [language model](#language-model) indicating it wants to invoke a named external function. In Tryll, tool calls are **detected only** — the server parses them from model output and returns them to the client, which is responsible for execution. See the [tool calling concept page](https://docs.tryllengine.com/concepts/tool-calling.html).

### Tool-call format

The specific textual convention the [language model](#language-model) uses to express a [tool call](#tool-call). Tryll supports multiple families (Llama-style, ChatML-style, JSON, XML); the active format is configured on the [tool-call node](https://docs.tryllengine.com/reference/nodes/tool-call.html).

______________________________________________________________________

## Constrained decoding

### Constrained decoding

Forcing a [language model](#language-model)'s output to match a formal [grammar](#gbnf) at every decode step: tokens that could not legally continue the output are masked out before sampling, so malformed output is impossible (not merely discouraged). Also called grammar-constrained generation. In Tryll it is exposed via the `grammar` param on [Generate](https://docs.tryllengine.com/reference/nodes/generate.html) and [Generate and Speak](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html) nodes. It guarantees the output's *shape*, never its *meaning* — a constrained model can still emit a valid-but-wrong answer. See the [constrained output concept page](https://docs.tryllengine.com/concepts/constrained-output.html).

### GBNF

GGML BNF — the grammar notation used by `llama.cpp` (and therefore by Tryll's `grammar` param) to describe [constrained](#constrained-decoding) output. A grammar is a set of `name ::= …` rules over string literals, alternation (`|`), and sequences; it must define a rule named `root` and must not be left-recursive. See the [upstream GBNF guide](https://github.com/ggml-org/llama.cpp/blob/master/grammars/README.md) for the full syntax.

______________________________________________________________________

## Transport

### Wire protocol

The byte-level contract between the Tryll client and server: TCP transport, 4-byte little-endian length prefix, FlatBuffers payload, 1 MiB max frame. See the [wire protocol reference](https://docs.tryllengine.com/reference/wire-protocol.html).

______________________________________________________________________

## Editor tools

### Chat Window

An editor-only window (`Window ▸ Tryll ▸ Chat`) that creates its own agent from a scene component and lets you converse with it without entering Play mode. Hosts the [turn inspector](#turn-inspector) in a side pane. See [Test an agent in the editor](https://docs.tryllengine.com/how-to/test-agent-in-editor.html).

### Dialog Lab

An editor-only window that runs one scripted [dialog](#dialog) across parameter variants and seeds, then shows every result side by side so the spread is visible. Makes no quality claim — it shows raw output, deterministic checks, and timings. Ships in both editors. See [Compare dialog variants](https://docs.tryllengine.com/how-to/compare-dialog-variants.html).

### Agent Log

An editor-only window that records what this editor process sent to and received from each [agent](#agent) — turns, voice auto-send, deferred variable flushes, and lifecycle events — as a per-agent narrative rather than an event dump. Scoped to **one client connection**, not the whole server. See [Use the Agent Log](https://docs.tryllengine.com/how-to/use-agent-log.html).

### Turn inspector

The shared panel that explains a single [turn](#turn): which [nodes](#node) ran and via which [exit routes](#exit-route), per-node timing, the node parameters used, the rendered [prompt](#prompt) and output, [retrieval](#retrieval) hits, and [tool calls](#tool-call). One implementation, hosted by the [Chat Window](#chat-window), [Agent Log](#agent-log), and [Dialog Lab](#dialog-lab). See the [turn inspector reference](https://docs.tryllengine.com/reference/turn-inspector.html).

### Turn diagnostics

The per-turn `debug_info` payload the server attaches when an [agent](#agent) was created with `enable_diagnostics`. It is the data the [turn inspector](#turn-inspector) renders. The flag is **structural** — it cannot be enabled on an agent that already exists.

### Server monitor

A contributor-facing browser UI served by a dev-time HTTP/SSE sidecar in the server process. Sees every [session](#session) on the server, which is its advantage over the [Agent Log](#agent-log); it cannot see what a client sent, which is the Agent Log's. Compiled out of Production builds.

______________________________________________________________________

See also:

- [Concept map](https://docs.tryllengine.com/concepts/index.html) — narrative explanations of the terms above.
- [Error codes](https://docs.tryllengine.com/reference/error-codes.html) — numeric error catalog used across all responses.
