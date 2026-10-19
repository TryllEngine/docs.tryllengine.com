# Workflow Nodes

A Tryll [workflow](https://docs.tryllengine.com/v0.9/reference/glossary.html#workflow) is a graph of nodes, each responsible for one step of the per-turn loop. Sixteen node types ship in the box; together they cover the common patterns of conversational inference (generation, retrieval, intent classification, instructions, detection, fallback, guardrails, composition, routing).

Every producing node below writes its text output into a named **slot** on the turn's shared blackboard, consumed by downstream nodes via `input` or by any template's `{{slot.<name>}}` tag — see [Slots and inter-node value passing](https://docs.tryllengine.com/v0.9/concepts/slots-and-inter-node-value-passing.html).

Nodes are referenced by their `NodeType` name on the [wire protocol](https://docs.tryllengine.com/v0.9/reference/wire-protocol.html):

| Node                                                                                                  | `NodeType`            | Exit routes                     | Use for                                                                                                                                                                                       |
| ----------------------------------------------------------------------------------------------------- | --------------------- | ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Generate](https://docs.tryllengine.com/v0.9/reference/nodes/generate.html)                           | `Generate`            | `default`                       | Language-model inference. The workhorse.                                                                                                                                                      |
| [Generate and Speak](https://docs.tryllengine.com/v0.9/reference/nodes/generate-and-speak.html)       | `GenerateAndSpeak`    | `default`                       | Fused LLM generation + streaming TTS.                                                                                                                                                         |
| [Speak](https://docs.tryllengine.com/v0.9/reference/nodes/speak.html)                                 | `Speak`               | `default`                       | TTS-only; voices its `input`-resolved slot (default: `user_message`, like every other input-bearing node).                                                                                    |
| [Transform](https://docs.tryllengine.com/v0.9/reference/nodes/transform.html)                         | `Transform`           | `default`                       | Model-free: renders a Mustache template into its own slot. No inference, no wire emission — composition/rewriting glue between other nodes.                                                   |
| [Instruction](https://docs.tryllengine.com/v0.9/reference/nodes/instruction.html)                     | `Instruction`         | `default`                       | Attach a named instruction slot to the current interaction; rendered by downstream `Generate` via `{{#instructions}}` / `{{slot.<name>}}`.                                                    |
| [Retrieve](https://docs.tryllengine.com/v0.9/reference/nodes/retrieve.html)                           | `Retrieve`            | `found`, `not_found`            | Vector search over a static embedded string storage or an experimental named mutable KB.                                                                                                      |
| [Write to Memory](https://docs.tryllengine.com/v0.9/reference/nodes/write-to-memory.html)             | `WriteToMemory`       | `written`, `failed`             | Experimental upsert into a named mutable KB. Place Retrieve **before** write in the same turn.                                                                                                |
| [Classify Intent](https://docs.tryllengine.com/v0.9/reference/nodes/classify-intent.html)             | `ClassifyIntent`      | `found`, `not_found`            | Top-1 embedding similarity against a labelled KB; attaches an `IntentionComponent` for downstream routing.                                                                                    |
| [Classify Intent (LLM)](https://docs.tryllengine.com/v0.9/reference/nodes/classify-intent-llm.html)   | `ClassifyIntentLLM`   | `found`, `not_found`            | Intent classification via the language model's first-token logprobs.                                                                                                                          |
| [Intent to Instruction](https://docs.tryllengine.com/v0.9/reference/nodes/intent-to-instruction.html) | `IntentToInstruction` | `default`                       | Look up the detected intent in a `Map` string storage and write the matching instruction slot.                                                                                                |
| [Tool Call](https://docs.tryllengine.com/v0.9/reference/nodes/tool-call.html)                         | `ToolCall`            | `tool_called`, `no_tool_called` | Detect whether the model wants to call an external tool.                                                                                                                                      |
| [Canned Response](https://docs.tryllengine.com/v0.9/reference/nodes/canned-response.html)             | `CannedResponse`      | `default`                       | Emit a fixed reply from a [string storage](https://docs.tryllengine.com/v0.9/reference/string-storage.html) without calling the model.                                                        |
| [Regex Guardrail](https://docs.tryllengine.com/v0.9/reference/nodes/regex-guardrail.html)             | `RegexGuardrail`      | `triggered`, `not_triggered`    | Regex-match an input slot (default `user_message`); branch the graph on pattern hits.                                                                                                         |
| [Pause](https://docs.tryllengine.com/v0.9/reference/nodes/pause.html)                                 | `Pause`               | `default`, `timeout`            | No-op checkpoint; the executor suspends the turn after this node until the client sends `ResumeAgentRequest`, or until `pause_timeout_ms` elapses (`0` waits forever).                        |
| [Branch](https://docs.tryllengine.com/v0.9/reference/nodes/branch.html)                               | `Branch`              | `then`, `else`                  | Model-free conditional routing on a turn-local slot or an agent variable — the only node that can read what another node produced *this turn*.                                                |
| [Summarize](https://docs.tryllengine.com/v0.9/reference/nodes/summarize.html)                         | `Summarize`           | `default`                       | Fold ageing turns into a rolling summary that Generate-class nodes render in their system message, so a long conversation stays coherent at a fixed token cost instead of being trimmed away. |

## Exit-route legend

Every node's reference page lists its named exits. Each exit is a typed string field on the node's params object (e.g. `default_exit`, `triggered_exit`). An **empty string** routes the turn to END (the default); a **non-empty string** must name another node in the graph. The server rejects graphs where an exit field names a missing node with error [`3003 GraphCompilationFailed`](https://docs.tryllengine.com/v0.9/reference/error-codes.html#3xxx-agent).

Common patterns:

- **Straight line.** Generate → `default` → `END`. Minimal single-turn agent.
- **Branch on guardrail.** Guardrail → `triggered` → CannedResponse → `END`; Guardrail → `not_triggered` → Generate → `default` → `END`.
- **Instruction then generate.** Instruction → `default` → Generate. The instruction's text is exposed as `{{slot.<name>}}` and `{{#instructions}}` in the Generate node's Mustache template.
- **Rewrite then retrieve.** Transform → `default` → Retrieve (`input` set to the Transform node's name) → Generate. See [Query rewriting for RAG](https://docs.tryllengine.com/v0.9/how-to/query-rewriting-for-rag.html).
- **Retrieve then generate.** Retrieve → `found` → Generate; Retrieve → `not_found` → Generate (same target). The `not_found` path is fine — the projection drops to the user message alone.
- **Tool-call detection.** ToolCall → `tool_called` → `END` (client handles the tool); ToolCall → `no_tool_called` → Generate.
- **Intent classification.** ClassifyIntent → `found` → IntentToInstruction → `default` → Generate; ClassifyIntent → `not_found` → Generate (general answer, no instruction override). See [How to build an intent-driven NPC](https://docs.tryllengine.com/v0.9/how-to/build-intent-driven-npc.html).
- **Refusal fallback.** Generate (hidden classification pass, `send=None`) → `default` → Branch (`Contains`/`Equals` on the verdict slot) → `then` → CannedResponse → `END`; Branch → `else` → Generate (the real answer) → `default` → `END`. Catches a model's own "I can't help with that" or an unsafe verdict without shipping the refusal text verbatim.

## Params

Every node carries one typed params table, an arm of the `NodeParams` FlatBuffers union in `nodes.fbs` (for example `GenerateParams` or `RetrieveParams`). A `NodeDescription` is just a `name` plus those `params`; there is no `(key, value)` parameter list. Each node's reference page lists the fields of its table, their defaults and ranges, and which ones are structural. Tool definitions live on `ToolCallParams.tools`.

On the wire, the union discriminator is 1-based and follows the order of the `NodeParams` union (`GenerateParams` = 1 … `SummarizeParams` = 16). Client `NodeType` enums are 0-based, so a client enum value is one lower than the wire discriminator for the same node.

## Diagnostics

When the agent is created with `enable_diagnostics = true`, every node contributes to `TurnComplete.debug_info`. Each node's reference page has a **Diagnostics** section listing the keys it records; the payload that holds them — the envelope, the shared prompt and engine sections, and the rules for when a key is omitted — is documented in [Turn Diagnostics JSON](https://docs.tryllengine.com/v0.9/reference/turn-diagnostics-json.html). See also [agent parameters — diagnostics](https://docs.tryllengine.com/v0.9/reference/agent-parameters.html#diagnostics).

## Related

- [Concept: Workflows, graphs, and nodes](https://docs.tryllengine.com/v0.9/concepts/workflows-graphs-nodes.html)
- [Agent Parameters](https://docs.tryllengine.com/v0.9/reference/agent-parameters.html) — how the graph is carried on the wire.
- [Glossary](https://docs.tryllengine.com/v0.9/reference/glossary.html#node)
