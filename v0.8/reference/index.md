# Reference

Complete, declarative documentation of every Tryll surface. Use this section when you need an authoritative answer about what a field is, what values it accepts, or what a specific error code means. For conceptual background, see the [Concepts](https://docs.tryllengine.com/v0.8/concepts/index.html) section; for task-driven walkthroughs, see [How-to](https://docs.tryllengine.com/v0.8/how-to/index.html).

It splits in two: **[Client API](#client-api)** is what your game code calls, **[Workflows and data](#workflows-and-data)** is what you declare for the server to execute.

## Client API

The surface your game code touches — one page per platform, plus the request fields, diagnostics, and errors shared by all of them.

### Per platform

- [Unity](https://docs.tryllengine.com/v0.8/reference/client-api/unity/index.html) — the `com.tryll.client` package: `TryllClient`, `TryllAgentComponent`, `TryllWorkflowAsset`, `TryllRuntimeSettings`, `TryllSpeaker`, `TryllVoiceInputComponent`.
- [Unreal C++](https://docs.tryllengine.com/v0.8/reference/client-api/unreal/cpp/index.html) — `TryllClient` plugin headers: `UTryllSubsystem`, `FTryllAgent`, and friends.
- [Unreal Blueprint](https://docs.tryllengine.com/v0.8/reference/client-api/unreal/blueprint-catalog.html) — every Blueprint-exposed UFUNCTION.
- [C++](https://docs.tryllengine.com/v0.8/reference/client-api/cpp/index.html) — `Tryll::Client::*`.
- [Python](https://docs.tryllengine.com/v0.8/reference/client-api/python/index.html) — `tryll_client`.

### Shared across platforms

- [Agent Parameters](https://docs.tryllengine.com/v0.8/reference/agent-parameters.html) — every field on `CreateAgentRequest`: graph, variables, diagnostics, knowledge presentation, KV-cache initialization, workload, tools.
- [Turn Inspector](https://docs.tryllengine.com/v0.8/reference/turn-inspector.html) — the shared per-turn diagnostics panel: every block, when each appears, and what to do when one is missing. Hosted by the Chat Window, Agent Log, and Dialog Lab.
- [Turn Diagnostics JSON](https://docs.tryllengine.com/v0.8/reference/turn-diagnostics-json.html) — every field of the `debug_info` payload behind that panel, and behind `TurnComplete` on the wire: the envelope, the per-node sections, and why a key is absent.
- [Error Codes](https://docs.tryllengine.com/v0.8/reference/error-codes.html) — numeric catalog, ranges, per-code recovery guidance.

## Workflows and data

What you declare, and what the server does with it.

### Workflow nodes

[Node catalog](https://docs.tryllengine.com/v0.8/reference/nodes/index.html) — the table of every node type, its exits, and whether it needs a model. Start there when choosing a node.

| Node                                                                                                  | What it does                                                                |
| ----------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| [Generate](https://docs.tryllengine.com/v0.8/reference/nodes/generate.html)                           | Run a language model on the projected dialogue and emit its answer.         |
| [Generate and Speak](https://docs.tryllengine.com/v0.8/reference/nodes/generate-and-speak.html)       | Generate and synthesize speech in one fused streaming step.                 |
| [Speak](https://docs.tryllengine.com/v0.8/reference/nodes/speak.html)                                 | TTS-only — voice text another node already produced.                        |
| [Instruction](https://docs.tryllengine.com/v0.8/reference/nodes/instruction.html)                     | Inject a static instruction via the `{{#instructions}}` block.              |
| [Transform](https://docs.tryllengine.com/v0.8/reference/nodes/transform.html)                         | Model-free Mustache rendering into a slot; emits nothing on the wire.       |
| [Branch](https://docs.tryllengine.com/v0.8/reference/nodes/branch.html)                               | Model-free conditional routing on a slot or agent variable.                 |
| [Pause](https://docs.tryllengine.com/v0.8/reference/nodes/pause.html)                                 | Suspend the turn between nodes and hand control back to the client.         |
| [Retrieve](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve.html)                           | Search an embedded string storage and attach the hits as knowledge.         |
| [Filter grammar](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve-filter-grammar.html)      | The JSON predicate grammar for metadata-filtered retrieval.                 |
| [Classify Intent](https://docs.tryllengine.com/v0.8/reference/nodes/classify-intent.html)             | Label the input by top-1 embedding similarity; exits `found` / `not_found`. |
| [Classify Intent (LLM)](https://docs.tryllengine.com/v0.8/reference/nodes/classify-intent-llm.html)   | Label the input by first-token logprob on a small model.                    |
| [Intent to Instruction](https://docs.tryllengine.com/v0.8/reference/nodes/intent-to-instruction.html) | Map an intent label to an instruction through a `Map`-kind storage.         |
| [Tool Call](https://docs.tryllengine.com/v0.8/reference/nodes/tool-call.html)                         | Advertise tools, let the model call them, control notify/pause/results.     |
| [Canned Response](https://docs.tryllengine.com/v0.8/reference/nodes/canned-response.html)             | Emit a fixed reply from a string storage — no model involved.               |
| [Regex Guardrail](https://docs.tryllengine.com/v0.8/reference/nodes/regex-guardrail.html)             | Match input against patterns and route away before spending a model call.   |

### Knowledge and strings

- [String Storage](https://docs.tryllengine.com/v0.8/reference/string-storage.html) — session-owned name-to-strings containers (`List` / `Map` / `Multimap`) used by canned-response, guardrail, and intent nodes.
- [Embedded String Storage](https://docs.tryllengine.com/v0.8/reference/embedded-string-storage.html) — vector-indexed knowledge bases used by retrieve and intent nodes.

### Models

- [Model Management](https://docs.tryllengine.com/v0.8/reference/model-management.html) — `models.json` schema, variants, download and load lifecycle, model status.
- [TTS Models](https://docs.tryllengine.com/v0.8/reference/tts-models.html) — shipped voices (Sherpa-ONNX and llama.cpp SNAC), which families Tryll supports, and why the rest cannot load.

### Server

- [Server Configuration](https://docs.tryllengine.com/v0.8/reference/server-configuration.html) — every field of `server-config.json`.
- [Telemetry](https://docs.tryllengine.com/v0.8/reference/telemetry.html) — what usage telemetry collects, and how to turn it off.
- [Wire Protocol](https://docs.tryllengine.com/v0.8/reference/wire-protocol.html) — TCP framing, FlatBuffers schema, session flow, `ErrorResponse` shape. The contract a third-party client must implement.

## Glossary

- [Glossary](https://docs.tryllengine.com/v0.8/reference/glossary.html) — canonical definitions for every term used across the documentation. Check here before guessing what a word means.
