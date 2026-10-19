# Change Agent Parameters at Runtime

Adjust a node's configuration while the agent is idle — for example to switch personalities by updating `system_prompt`, or to tune retrieval precision by changing `top_k` — without recreating the agent.

Prerequisites

- An agent created and ready to receive messages — see [Build a Chat Agent with a Graph](https://docs.tryllengine.com/v0.9/how-to/build-chat-agent-with-graph.html).
- The agent must be idle — or **paused** — when you call `ChangeParams` / `change_params`. The server rejects the request with error `3004 AgentBusy` if a turn is actively running. See [How to pause and resume a turn](https://docs.tryllengine.com/v0.9/how-to/pause-and-resume-turns.html) for the one exception: a turn suspended at a `Pause` node (or a paused `ToolCall`) allows `ChangeParams` / `change_params` while it's parked.

______________________________________________________________________

## How it works

Parameters are now **typed** — each node type has a dedicated params object (e.g. `GenerateParams`, `RetrieveParams`) instead of a string key/value map.

The canonical mutation pattern is **clone-set-send**:

1. **Clone** the baseline params object (preserves structural/immutable fields automatically).
1. **Set** only the fields you want to change.
1. **Send** the modified object via `ChangeParams`.

The server diffs the supplied params against the node's current params, vetoes any structural fields that have changed (`3006 ParamNotMutable`), validates values (`3007 InvalidParamValue`), and applies the delta.

______________________________________________________________________

## Which fields can be changed?

Fields marked `(structural)` in the schema **cannot** be changed at runtime. All other fields are mutable unless a node-specific validator rejects the value.

Structural on every node

On every node that has them, the exit fields (`default_exit`, `found_exit`, `not_found_exit`, `tool_called_exit`, `no_tool_called_exit`, `timeout_exit`, …), plus `context_size`, `input`, and `output_name`, are **structural** — fixed at `CreateAgent`. So is `adaptation` (the [LoRA adapters](https://docs.tryllengine.com/v0.9/how-to/use-a-lora-adapter.html)) on every LLM node. The table lists only the node-specific structural fields *in addition* to those.

| Node                  | Node-specific structural fields                      | Mutable fields                                                                                                                                                                        |
| --------------------- | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Generate`            | `model_name`, `context`                              | `system_prompt`, `template`, `placement`, `send`, `history_role`, `grammar`, `substitute_agent_variables`, `output_filter.*`, all `sampling.*`                                        |
| `GenerateAndSpeak`    | `model_name`, `context`                              | `system_prompt`, `template`, `placement`, `history_role`, `grammar`, `substitute_agent_variables`, `output_filter.*`, `min_sentence_chars`, `first_chunk_max_chars`, all `sampling.*` |
| `Speak`               | —                                                    | `min_sentence_chars`, `first_chunk_max_chars`                                                                                                                                         |
| `ToolCall`            | `model_name`, `tools`, `pause_timeout_ms`, `context` | `system_prompt`, `mode`, `parallel_tool_calls`, `disposition`, `acknowledge_text`, all `sampling.*`                                                                                   |
| `Retrieve`            | `embedded_string_storage`                            | `top_k`, `threshold`, `source`, `filter`                                                                                                                                              |
| `CannedResponse`      | `inline_strings`                                     | `string_storage` (rebind), `selection_strategy`, `send`, `history_role`                                                                                                               |
| `Transform`           | —                                                    | `template`                                                                                                                                                                            |
| `Pause`               | `pause_timeout_ms`                                   | *(none — Pause has no mutable params)*                                                                                                                                                |
| `RegexGuardrail`      | `inline_strings`                                     | `string_storage` (rebind)                                                                                                                                                             |
| `Instruction`         | —                                                    | `instruction`                                                                                                                                                                         |
| `ClassifyIntent`      | `embedded_string_storage`                            | `metadata_field`, `threshold`, `filter`, `notify_client`, `diagnostic_topk`                                                                                                           |
| `ClassifyIntentLLM`   | `model_name`, `intents`, `output_name`               | `system_prompt`, `history_turns`, `threshold`, `margin`, `notify_client`, `not_found_intent`                                                                                          |
| `IntentToInstruction` | `inline_keys`, `inline_strings`                      | `string_storage` (rebind)                                                                                                                                                             |

`tool_call_format` on `ToolCall` is deprecated and ignored by the server (tool prompting is chat-template driven), so it is not shown; do not set it.

The voice is not a node parameter

Since protocol v27 the TTS model, speaker, speed, language, reference clip, voice description, delivery and `emit_visemes` live on the agent's [voice profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html), not on `Speak` or `GenerateAndSpeak`, so `ChangeAgentParam` cannot change them. Use `SetAgentVoice` to replace the voice (idle only), or `SetAgentDeliveryMode` to switch its delivery mode (any time; applies from the next sentence). See [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.9/how-to/add-voice-to-an-agent.html#8-change-the-voice-or-its-delivery-at-runtime).

______________________________________________________________________

## Examples

```
using Tryll.Client;

var agentComp = GetComponent<TryllAgentComponent>();

// Clone the baseline, change one field, send.
// GetNodeParamsBaseline already returns a deep copy — mutate in place.
var p = (TryllGenerateParams)agentComp.GetNodeParamsBaseline("answer");
p.SystemPrompt = "You are a pirate.";
agentComp.ChangeParams("answer", p,
    err => {
        if (err.IsOk)
            Debug.Log("Persona updated");
        else
            Debug.LogError($"Error {err.Code}: {err.Message}");
    });
// Or bind OnParamChanged in the Inspector / AddListener.

// Relax the retrieval threshold.
var rp = (TryllRetrieveParams)agentComp.GetNodeParamsBaseline("knowledge");
rp.Threshold = 0.9f;
agentComp.ChangeParams("knowledge", rp);

// Switch to a single whole-text answer instead of streamed deltas.
var gp = (TryllGenerateParams)agentComp.GetNodeParamsBaseline("answer");
gp.Send = TryllSendAnswer.Whole;
agentComp.ChangeParams("answer", gp);
```

```
#include "Generated/Nodes/TryllGenerateParams.h"
#include "Generated/Nodes/TryllNodeParamsFactory.h"

// Clone the baseline, change one field, send.
UTryllNodeParamsBase* Base = AgentComponent->GetNodeParamsBaseline(TEXT("answer"));
UTryllGenerateParams* P = Cast<UTryllGenerateParams>(
    UTryllNodeParamsFactory::CloneParams(Base, this));
P->SystemPrompt = TEXT("You are a pirate.");
AgentComponent->ChangeParams(
    TEXT("answer"), P,
    [](const FTryllError& Err)
    {
        if (Err.Code == 0)
            UE_LOG(LogTemp, Log, TEXT("Persona updated"));
        else
            UE_LOG(LogTemp, Warning, TEXT("Error %d: %s"), Err.Code, *Err.Message);
    });

// Relax the retrieval threshold.
UTryllRetrieveParams* RP = Cast<UTryllRetrieveParams>(
    UTryllNodeParamsFactory::CloneParams(
        AgentComponent->GetNodeParamsBaseline(TEXT("knowledge")), this));
RP->Threshold = 0.9f;
AgentComponent->ChangeParams(TEXT("knowledge"), RP);
```

1. Call **Get Node Params Baseline** on the `TryllAgentComponent` to get the current params object for a node.
1. Call **Clone Params** (from `TryllNodeParamsFactory`) on the result.
1. Set any fields on the cloned object.
1. Call **Change Params** on the `TryllAgentComponent` and bind **On Param Changed** to handle success or error.

The C++ client does not cache a baseline copy of authored params. Keep your own copy of the params you last sent (or authored at `CreateAgent`) and start each mutation from that local copy. Structural fields must match the create-time values or the server returns `ParamNotMutable`.

```
#include <tryll/AgentProxy.h>
// XxxT PODs come from the flatc-generated messages_generated.h (--gen-object-api)
#include <NodeParams_generated.h>

using namespace Tryll::NodeParams;
using namespace Tryll::Client;

// Clone the caller-owned baseline, change system prompt, send asynchronously.
GenerateParamsT p = answerBaseline;          // caller-owned copy
p.system_prompt = "You are a pirate.";
agent.ChangeParamsAsync("answer", std::move(p)).get(); // or .then(...)

// Relax the retrieval threshold (synchronous).
RetrieveParamsT rp = knowledgeBaseline;      // caller-owned copy
rp.threshold = 0.9f;
agent.ChangeParams("knowledge", std::move(rp));
```

The Python client does not cache a baseline copy of authored params. Keep your own copy of the params you last sent (or authored at `create_agent`) and start each mutation from a `deepcopy` of that local copy.

```
from copy import deepcopy
from tryll_client.graph import GenerateParams, RetrieveParams, SendAnswer

# Clone the caller-owned baseline, change system_prompt, send.
p = deepcopy(answer_baseline)        # caller-owned copy
p.system_prompt = "You are a pirate."
agent.change_params("answer", p)

# Relax the retrieval threshold.
rp = deepcopy(knowledge_baseline)
rp.threshold = 0.9
agent.change_params("knowledge", rp)

# Switch to a single whole-text answer instead of streamed deltas.
gp = deepcopy(answer_baseline)
gp.send = SendAnswer.Whole
agent.change_params("answer", gp)

# Flip a node between a strict "command turn" and free chat by setting or
# clearing its GBNF grammar (see the Constrained Output concept). The
# sampler rebuilds the grammar each turn, so the change takes effect next send.
gp = deepcopy(answer_baseline)
gp.grammar = 'root ::= "yes" | "no"'   # constrain; "" clears back to free chat
agent.change_params("answer", gp)
```

`change_params` raises a `TryllError` on failure (unknown node, structural field changed, or invalid value). A malformed GBNF grammar is rejected with `InvalidParamValue` (3007) and the previous grammar is kept.

______________________________________________________________________

## `system_prompt` and KV-cache rewind

Changing `system_prompt` does **not** immediately flush the KV cache. The change is stored and picked up on the *next* `send_message` call. At that point the projection pipeline detects the new token sequence, trims the stale tail of the cache, and re-decodes only the changed prefix. This is the most efficient path: back-to-back `change_params` / `send_message` calls impose only one re-decode, not two.

______________________________________________________________________

## Error codes

| Code   | Name                | Cause                                                                                                                          |
| ------ | ------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `3004` | `AgentBusy`         | A turn is actively running (not idle and not paused). Wait for `TurnComplete`, or call `ChangeParams` while parked at a pause. |
| `3005` | `UnknownNode`       | `node_name` does not match any node instance name in the graph.                                                                |
| `3006` | `ParamNotMutable`   | A structural field was included in the mutation request.                                                                       |
| `3007` | `InvalidParamValue` | A mutable field failed schema-range or node-specific validation.                                                               |

______________________________________________________________________

## Related

- [How to pause and resume a turn](https://docs.tryllengine.com/v0.9/how-to/pause-and-resume-turns.html)
- [How to constrain output with a grammar](https://docs.tryllengine.com/v0.9/how-to/constrain-output-with-a-grammar.html)
- [Concept: Constrained output (GBNF)](https://docs.tryllengine.com/v0.9/concepts/constrained-output.html)
- [Generate node — parameters](https://docs.tryllengine.com/v0.9/reference/nodes/generate.html#parameters)
- [Tool Call node — parameters](https://docs.tryllengine.com/v0.9/reference/nodes/tool-call.html#parameters)
- [Retrieve node — parameters](https://docs.tryllengine.com/v0.9/reference/nodes/retrieve.html#parameters)
- [Canned Response node — parameters](https://docs.tryllengine.com/v0.9/reference/nodes/canned-response.html#parameters)
- [Regex Guardrail node — parameters](https://docs.tryllengine.com/v0.9/reference/nodes/regex-guardrail.html#parameters)
- [Build a Chat Agent with a Graph](https://docs.tryllengine.com/v0.9/how-to/build-chat-agent-with-graph.html)
