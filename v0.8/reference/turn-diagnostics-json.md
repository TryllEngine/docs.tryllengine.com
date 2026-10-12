# Turn Diagnostics JSON

When an agent is created with `enable_diagnostics = true`, every `TurnComplete` carries a JSON string in `debug_info` describing what the turn actually did: which nodes ran, what prompt each model received, what it produced, and what the engine was doing underneath.

You meet this payload in two places, and they are the same bytes:

- **The Raw JSON block** at the bottom of the [turn inspector](https://docs.tryllengine.com/v0.8/reference/turn-inspector.html), in the Chat Window, the Agent Log and Dialog Lab. The editors pretty-print `debug_info` verbatim — they add no keys, rename none, and drop none.
- **`TurnComplete.debug_info`** on the wire, for Python, C++ and any other client. There is no panel there; this page is the whole story.

This page is the field reference. [Turn Inspector](https://docs.tryllengine.com/v0.8/reference/turn-inspector.html) documents the rendered view over the same data.

This is a diagnostic, not a contract

`debug_info` exists to be read by a human who is debugging. Its shape may change in any release, signalled by `schema_version`. Do not parse it in shipped game code — everything your game needs at runtime is on the typed wire messages. Tooling that does parse it should check `schema_version` and tolerate unknown and missing keys.

______________________________________________________________________

## Where to look for…

| Question                                            | Look at                                                                                                                                                                                                                                                                                                                   |
| --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The prompt was not what I expected                  | `nodes[].diagnostics.input.prompt[]`                                                                                                                                                                                                                                                                                      |
| Which node decided the route                        | `nodes[]._type`, `nodes[].name`, `nodes[].exit_route`                                                                                                                                                                                                                                                                     |
| Why the turn was slow                               | `time_to_first_token_ms`, `nodes[].duration_s`, `…diagnostics.engine.scheduler`; for voice, `…diagnostics.tts.time_to_first_audio_ms` (Speak: `…parameters.time_to_first_audio_ms`)                                                                                                                                       |
| Why the model re-processed the whole prompt         | `engine.scheduler.last_sync.prefill.tokens_reused` vs `tokens_decoded`                                                                                                                                                                                                                                                    |
| Whether classification matched, and against what    | `nodes[].diagnostics.output.classification` — `winner` / `intent` on `found`; `rejected_intent` (and embedding `rejected_distance`) on `not_found` when a pick existed. LLM `probs[i]` is the probability of `parameters.intents_ids[i]` (equal length on `found`). The query is `input.query` for both classifier kinds. |
| What the retriever found, including near-misses     | `nodes[].diagnostics.output.results[]`                                                                                                                                                                                                                                                                                    |
| Which `__VARIABLE__` markers were substituted       | `nodes[].variable_replacements[]`                                                                                                                                                                                                                                                                                         |
| What the output filter deleted from the answer      | `nodes[].output_filter_removals[]`                                                                                                                                                                                                                                                                                        |
| Whether the tool grammar was actually applied       | `engine.chat_turn.has_grammar`, `grammar_lazy`                                                                                                                                                                                                                                                                            |
| What the turn's slots and knowledge held at the end | `interaction.components[]`                                                                                                                                                                                                                                                                                                |
| Why the turn ended early                            | `status`, `errors[]`                                                                                                                                                                                                                                                                                                      |

______________________________________________________________________

## Envelope

The top level of `debug_info`. The first nine keys are always present, in this order. `interaction` is appended after them only when the server enables it.

| Key                      | Type   | Meaning                                                                                                                                                                   |
| ------------------------ | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `schema_version`         | int    | Shape version of this payload. Currently `2`. A reader that does not recognise the value should fall back to showing raw text.                                            |
| `status`                 | string | Terminal status: `success`, `error`, `cancelled`, or `pause_timed_out`.                                                                                                   |
| `duration_s`             | number | Wall time for the whole turn, in seconds, including time spent paused. Subtract `paused_duration_s` to get engine time.                                                   |
| `time_to_first_token_ms` | number | Delay before the first streamed token reached the client. `0` when streaming was off or nothing was generated — this is the number that governs perceived responsiveness. |
| `paused_duration_s`      | number | Total time the turn spent paused, summed across every pause, whether it ended with a resume or a timeout. Included in `duration_s`. Excluded from per-node `duration_s`.  |
| `pause_count`            | int    | How many times the turn paused.                                                                                                                                           |
| `tool_calls`             | array  | Tool calls the turn emitted. See [Tool calls](#tool-calls). Empty array when none.                                                                                        |
| `nodes`                  | array  | One entry per **node execution**, in execution order. See [Node entries](#node-entries).                                                                                  |
| `errors`                 | array  | Errors reported during the turn. See [Errors](#errors). Empty array on success.                                                                                           |
| `interaction`            | object | End-of-turn snapshot of the turn's components. **Omitted** unless the server enables it — see [Why a key is absent](#why-a-key-is-absent).                                |

### Key ordering

The envelope and each node entry use a fixed declaration order (the tables here match it). Inside a node's `diagnostics` document — and inside each `interaction` component — the keys are **sorted alphabetically**. That is why `engine` comes before `input`, `output` and `parameters` even though the engine section is written last, and why `_type` always leads a component.

______________________________________________________________________

## Node entries

`nodes[]` holds one entry per node *execution*. A node the graph enters twice appears twice, in the order it ran.

| Key                                | Type   | Meaning                                                                                                                                            |
| ---------------------------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `_type`                            | string | Node class name — `GenerateNode`, `RetrieveNode`, `BranchNode`, … Note the leading underscore.                                                     |
| `name`                             | string | The node's name in your graph.                                                                                                                     |
| `exit_route`                       | string | The exit this execution took (`default`, `found`, `not_found`, `triggered`, `then`, …).                                                            |
| `duration_s`                       | number | Time this execution took, in seconds. Model-free nodes are routinely in the microseconds.                                                          |
| `t_start_ms`, `t_end_ms`           | number | Turn-relative timestamps. **Never present in `debug_info`** — the telemetry pipeline sets them, the wire payload does not.                         |
| `diagnostics`                      | object | What the node itself recorded. See [Diagnostics sections](#diagnostics-sections). **Absent** for a node that records nothing (`PauseNode`).        |
| `variable_replacements`            | array  | Output-side `__VARIABLE__` substitutions this node performed. Absent unless the node ran with `substitute_agent_variables` and replaced something. |
| `variable_replacements_truncated`  | bool   | `true` when more than 256 replacements occurred and the array was cut. Present whenever `variable_replacements` is.                                |
| `output_filter_removals`           | array  | Text this node's `output_filter` deleted. Absent unless a filter rule fired.                                                                       |
| `output_filter_removals_truncated` | bool   | `true` when more than 256 removals occurred. Present whenever `output_filter_removals` is.                                                         |

A node execution with no `diagnostics` still carries its identity and timing:

```
{
  "_type": "PauseNode",
  "name": "checkpoint",
  "exit_route": "default",
  "duration_s": 3.5e-06
}
```

### `variable_replacements[]` and `output_filter_removals[]`

Both are node-level, not inside `diagnostics`, and both are visible **only** here — the turn inspector has no dedicated block for either.

| Key                  | Meaning                                                                                                             |
| -------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `variable`           | Canonical declared variable name.                                                                                   |
| `rule`               | Stable filter rule id — `strip_speaker_prefix`, `strip_asterisk_spans`, `strip_paren_spans`, `strip_bracket_spans`. |
| `matched`            | The exact text the model produced (the marker, or the removed span).                                                |
| `replacement`        | The value inserted. Replacements only.                                                                              |
| `source_byte_offset` | UTF-8 offset in the raw model stream.                                                                               |
| `output_byte_offset` | UTF-8 offset in the transformed stream the client received.                                                         |

```
"variable_replacements": [
  { "variable": "price", "matched": "__PrIcE__", "replacement": "18",
    "source_byte_offset": 11, "output_byte_offset": 11 }
],
"variable_replacements_truncated": false
```

```
"output_filter_removals": [
  { "rule": "strip_speaker_prefix", "matched": "Barnaby: ",
    "source_byte_offset": 0, "output_byte_offset": 0 },
  { "rule": "strip_asterisk_spans", "matched": "*sighs*",
    "source_byte_offset": 9, "output_byte_offset": 0 }
]
```

The two offsets diverge exactly where earlier edits shifted the stream — the second removal above starts at byte 9 of what the model wrote, but at byte 0 of what the player saw.

See [Substitute agent variables in output](https://docs.tryllengine.com/v0.8/how-to/substitute-agent-variables-in-output.html) and [Filter LLM output artifacts](https://docs.tryllengine.com/v0.8/how-to/filter-llm-output-artifacts.html).

______________________________________________________________________

## Diagnostics sections

Each node's `diagnostics` object is a small document assembled by that node. Five section names are in use:

| Section      | Contains                                                                                                                                        | Emitted by                                                               |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| `parameters` | The values the node actually ran with, after every fallback and override resolved.                                                              | Every node that records anything.                                        |
| `input`      | What went into the node — `prompt` for model nodes, `query` for retrieval and both classifier kinds (`ClassifyIntent` and `ClassifyIntentLLM`). | Nodes that feed a model or an index.                                     |
| `output`     | The node's semantic result — generated text, retrieval hits, classification verdict, routing decision.                                          | Most nodes.                                                              |
| `engine`     | Backend and scheduler counters. No user content.                                                                                                | Nodes that run a language model.                                         |
| `tts`        | Voice synthesis parameters, the sample rate, and `time_to_first_audio_ms` (measured from the node's start).                                     | `GenerateAndSpeak` only — `Speak` folds the same keys into `parameters`. |

The node reference pages list the keys each node contributes; the rest of this section covers what several nodes share. Start at the [node catalog](https://docs.tryllengine.com/v0.8/reference/nodes/index.html).

### `input.prompt[]` — the prompt as the model received it

Every node that runs a language model emits this, and it is usually the first thing worth reading. Projection has already happened: history replay, `{{#instructions}}`, knowledge blocks and Mustache substitution are all baked in.

| Key            | Meaning                                                                                                           |
| -------------- | ----------------------------------------------------------------------------------------------------------------- |
| `role`         | `system`, `user` or `assistant`.                                                                                  |
| `content`      | The message text as sent.                                                                                         |
| `tool_calls[]` | Present on historical assistant messages that called a tool: `id` (omitted when empty), `name`, `arguments_json`. |

```
"input": {
  "prompt": [
    { "role": "system", "content": "You are a knowledgeable aquarium expert…" },
    { "role": "system", "content": "Use the following information to answer the next question.\n\n- Goldfish (Carassius auratus) were selectively bred in ancient China…\n" },
    { "role": "user",   "content": "How long have goldfish been kept by humans, and where were they first bred?" }
  ]
}
```

If a prompt does not contain what you expected, this is where that shows up. See [Projection and token budgets](https://docs.tryllengine.com/v0.8/concepts/projection-and-token-budgets.html).

### Shared `parameters` keys

Nodes that sample from a language model all report the resolved sampling set: `temperature`, `top_p`, `top_k`, `max_tokens`, `min_p`, `seed`, `repeat_penalty`, `presence_penalty`, `frequency_penalty`. `GenerateAndSpeak` reports only the first four.

`model_name` is the **resolved** catalog name, after the node's `model_name` / `default_model_name` fallback — so it answers "which model actually ran", not "which one did I configure".

### `engine`

Present on language-model nodes, and removable by server configuration (see [below](#why-a-key-is-absent)). Nothing here is user content.

**`engine.chat_turn`** — the tool-calling and grammar state for the last prompt. Read it when a `ToolCall` node ran to `max_tokens` instead of calling anything: an empty or missing grammar means the sampler was unconstrained.

| Key                                            | Meaning                                                       |
| ---------------------------------------------- | ------------------------------------------------------------- |
| `has_turn`                                     | A chat turn was prepared.                                     |
| `has_grammar`                                  | A grammar was attached to the sampler.                        |
| `grammar_lazy`                                 | The grammar activates only once the model starts a tool call. |
| `grammar_bytes`                                | Size of the grammar. `0` means none.                          |
| `generation_prompt`, `generation_prompt_bytes` | The template's assistant-turn opener.                         |
| `tool_count`                                   | Tools advertised to the model.                                |
| `tool_choice`                                  | `auto`, `required` or `none`.                                 |

**`engine.scheduler`** — present only when the inference scheduler is enabled.

| Key                                                              | Meaning                                                                         |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| `last_job.kind`                                                  | `prefill`, `decode`, `score` or `generate`.                                     |
| `last_job.queue_wait_us`                                         | How long that job waited for a slot.                                            |
| `last_sync.prep_us`                                              | Time spent preparing the context before the backend ran.                        |
| `last_sync.prefill.backend_us`                                   | Backend time spent on prefill.                                                  |
| `last_sync.prefill.chunk_count`, `max_chunk_us`                  | Prefill chunking, and the slowest chunk.                                        |
| `last_sync.prefill.tokens_decoded`                               | Prompt tokens the model had to process.                                         |
| `last_sync.prefill.tokens_reused`                                | Prompt tokens served from the existing KV cache.                                |
| `totals.*`                                                       | Cumulative job counts and queue wait for this context.                          |
| `maxima.decode_queue_wait_us`                                    | Worst decode queue wait — contention with other agents.                         |
| `pacing.consumer_class`                                          | How this agent was classified: `interactive_visible`, `interactive_buffered`, … |
| `pacing.throttle_level`, `paced_delay_us`, `paced_quantum_count` | Manual throttle state and the delay it introduced.                              |

`tokens_reused` against `tokens_decoded` is the pair that explains most latency surprises: a turn that reused nothing re-processed the entire prompt. See [Manage an agent's KV cache](https://docs.tryllengine.com/v0.8/how-to/manage-agent-kv-cache.html) and [Run many agents on a small GPU](https://docs.tryllengine.com/v0.8/how-to/run-many-agents-on-a-small-gpu.html).

______________________________________________________________________

## Tool calls

The top-level `tool_calls[]` lists the calls the turn emitted, flattened for quick scanning:

```
"tool_calls": [
  { "name": "lookup_inventory", "params": { "query": "emeralds" } }
]
```

`params` is a flat string map of argument name to value. The richer view — the raw `arguments_json` the model produced, whether parsing fell back, and the result that came back — is on the `ToolCall` node's own `output` section and in `interaction`. See [Tool Call](https://docs.tryllengine.com/v0.8/reference/nodes/tool-call.html).

## Errors

`errors[]` carries one entry per error reported during the turn, in the order they fired.

| Key         | Meaning                                                                           |
| ----------- | --------------------------------------------------------------------------------- |
| `message`   | The error text.                                                                   |
| `node_name` | The node it was attributed to. Empty string when the error was not node-specific. |

An empty array with `status: "success"` is the normal case. `status` can be `cancelled` or `pause_timed_out` with `errors` still empty — the turn ended without anything going wrong.

## `interaction`

The end-of-turn snapshot of the turn's components: the slots, knowledge, intent and tool records the nodes wrote. Present only when the server enables it.

| Key            | Meaning                                                                                                |
| -------------- | ------------------------------------------------------------------------------------------------------ |
| `interrupted`  | The turn was interrupted before completing.                                                            |
| `components[]` | Every component on the interaction, each a **flat** map of string values with a `_type` discriminator. |

Component values are all strings, including numbers — `"distance": "0.2349"`, not `0.2349`.

A named value on the turn's blackboard. One per slot the turn wrote.

| Key            | Meaning                                                       |
| -------------- | ------------------------------------------------------------- |
| `name`         | Slot name.                                                    |
| `text`         | Its value.                                                    |
| `kind`         | `user`, `instruction` or `general`.                           |
| `history_role` | How it replays into later turns: `none`, `user`, `assistant`. |
| `producer`     | The node that wrote it.                                       |

Retrieved chunks attached by a `Retrieve` node. The chunk list is **flattened into keys** with a zero-padded index.

| Key                    | Meaning                                                                      |
| ---------------------- | ---------------------------------------------------------------------------- |
| `source`               | The `source` label from the Retrieve node. Omitted when empty.               |
| `chunks[000].id`       | Chunk id.                                                                    |
| `chunks[000].text`     | Chunk text.                                                                  |
| `chunks[000].distance` | Cosine distance to 4 decimals, or the string `"null"` for lexical-only hits. |

| Key      | Meaning                                       |
| -------- | --------------------------------------------- |
| `intent` | The intent label a classification node wrote. |

Written by a [`Summarize`](https://docs.tryllengine.com/v0.8/reference/nodes/summarize.html) node on the interaction that triggered a fold. Older summaries stay on earlier interactions and are inert: projection reads only the newest non-empty one.

| Key               | Meaning                                                      |
| ----------------- | ------------------------------------------------------------ |
| `text`            | The summary generated by this fold.                          |
| `covered_through` | Index of the last interaction folded into `text`, inclusive. |
| `producer`        | The Summarize node that wrote it.                            |

| Key           | Meaning                                               |
| ------------- | ----------------------------------------------------- |
| `tool_name`   | The function the model called.                        |
| `call_id`     | Correlates with the matching `ToolResultComponent`.   |
| `dispatched`  | `"true"` once the client was notified.                |
| `disposition` | The node's `disposition` for this call.               |
| `exchange_id` | Tool exchange this call belongs to. Omitted when `0`. |
| `model_id`    | The model's own call id. Omitted when empty.          |
| `arg.<name>`  | One key per argument.                                 |

The environment's answer to one call. Absent when the disposition omits history, or while `AwaitResult` is still waiting for the client.

| Key           | Meaning                             |
| ------------- | ----------------------------------- |
| `call_id`     | Links back to the `ToolCallRecord`. |
| `tool_name`   | The function that produced it.      |
| `text`        | The result text.                    |
| `exchange_id` | Omitted when `0`.                   |

______________________________________________________________________

## Why a key is absent

Most confusion about this payload is about something that is *not* in it. There are seven reasons, and they are all deliberate.

| What you see                       | Why                                                                                                                                                                                                                                                                                                                                                          |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| No `debug_info` at all             | The agent was created with `enable_diagnostics = false`. The flag is **structural** — it cannot be turned on for an agent that already exists. See [Agent Parameters](https://docs.tryllengine.com/v0.8/reference/agent-parameters.html#diagnostics) and the [turn inspector prerequisite](https://docs.tryllengine.com/v0.8/reference/turn-inspector.html). |
| No `engine` section on any node    | Server config `include_engine_diagnostics` is off. This is the **only** section a server flag removes from the wire payload.                                                                                                                                                                                                                                 |
| No `interaction` key               | Server config `include_interaction_in_diagnostics` is off. Tool-call *results* are joined through interaction components, so the inspector's tool-call block thins out too.                                                                                                                                                                                  |
| No `t_start_ms` / `t_end_ms`       | These are never in `debug_info`. Only the telemetry pipeline sets them.                                                                                                                                                                                                                                                                                      |
| A node has no `diagnostics` object | The node records nothing (`PauseNode`), or it recorded only optional values that were all empty.                                                                                                                                                                                                                                                             |
| An expected key is simply missing  | Optional fields are **omitted when empty rather than written as `null` or `0`** — an absent `filter` means no filter, an absent `bm25_score` means the sparse leg did not contribute. Absence is not zero.                                                                                                                                                   |
| An array looks short               | `variable_replacements` and `output_filter_removals` cap at 256 entries; check the matching `*_truncated` flag.                                                                                                                                                                                                                                              |

Both server flags live in [`server-config.json`](https://docs.tryllengine.com/v0.8/reference/server-configuration.html). The committed development config enables both; release staging turns both off, so a payload captured from a shipped build is thinner than one captured in the editor.

Missing Agent Log event ≠ missing diagnostics

`notify_client` on `Branch` and the classification nodes controls whether the *client* is told an event happened. It does not affect what those nodes write here. A classification that never appeared in the Agent Log is still fully described in this payload.

______________________________________________________________________

## Worked example

A two-node RAG turn — `Retrieve` → `Generate` — with long text trimmed. Every structural key is as the server emitted it.

```
{
  "schema_version": 2,
  "status": "success",
  "duration_s": 0.4519225,
  "time_to_first_token_ms": 125.0283,
  "paused_duration_s": 0,
  "pause_count": 0,
  "tool_calls": [],
  "nodes": [
    {
      "_type": "RetrieveNode",
      "name": "retrieve",
      "exit_route": "found",
      "duration_s": 0.0045927,
      "diagnostics": {
        "input": {
          "query": "How long have goldfish been kept by humans, and where were they first bred?"
        },
        "output": {
          "results": [
            { "distance": 0.23488157987594604, "id": "goldfish_1",
              "text": "Goldfish (Carassius auratus) were selectively bred in ancient China…" },
            { "distance": 0.39438164234161377, "id": "goldfish_4",
              "text": "Goldfish are cold-water fish that thrive at 65–72 °F (18–22 °C)…" }
          ]
        },
        "parameters": {
          "embedded_string_storage": "data/dbs/aquarium/aquarium_all_mini.json",
          "filter": "",
          "filtered_count": 1,
          "raw_result_count": 3,
          "result_count": 2,
          "retrieval_mode": "Dense",
          "rrf_k": 60,
          "source": "retrieve",
          "threshold": 0.4000000059604645,
          "top_k": 3
        }
      }
    },
    {
      "_type": "GenerateNode",
      "name": "generate",
      "exit_route": "default",
      "duration_s": 0.4472286,
      "diagnostics": {
        "engine": {
          "chat_turn": {
            "generation_prompt": "<|start_header_id|>assistant<|end_header_id|>\n\n",
            "generation_prompt_bytes": 47,
            "grammar_bytes": 0, "grammar_lazy": false,
            "has_grammar": false, "has_turn": true,
            "tool_choice": "auto", "tool_count": 0
          },
          "scheduler": {
            "enabled": true,
            "last_job": { "kind": "decode", "queue_wait_us": 5 },
            "last_sync": {
              "prefill": { "backend_us": 71677, "chunk_count": 2, "max_chunk_us": 69303,
                           "tokens_decoded": 355, "tokens_reused": 0 },
              "prep_us": 4179
            },
            "maxima": { "decode_queue_wait_us": 15, "paced_delay_us": 0 },
            "pacing": { "consumer_class": "interactive_visible", "paced_delay_us": 0,
                        "paced_quantum_count": 0, "throttle_level": 0 },
            "totals": { "completed_jobs": 45, "decode_jobs": 43, "generate_jobs": 0,
                        "prefill_jobs": 2, "queue_wait_us": 330, "score_jobs": 0 }
          }
        },
        "input": {
          "prompt": [
            { "role": "system", "content": "You are a knowledgeable aquarium expert assistant…" },
            { "role": "system", "content": "Use the following information to answer the next question.\n\n- Goldfish (Carassius auratus) were selectively bred in ancient China…\n" },
            { "role": "user",   "content": "How long have goldfish been kept by humans, and where were they first bred?" }
          ]
        },
        "output": {
          "text": "Goldfish have been kept by humans for over a thousand years…"
        },
        "parameters": {
          "frequency_penalty": 0,
          "history_role": "Assistant",
          "max_tokens": 2048,
          "min_p": 0.05000000074505806,
          "model_name": "Llama 3.2 3B Instruct (Q4_K_M)",
          "placement": "before_user_as_system",
          "presence_penalty": 0,
          "repeat_penalty": 1.2000000476837158,
          "seed": 42,
          "send": "Streamed",
          "temperature": 0,
          "template": "{{#knowledge}}{{#has_chunks}}Use the following information to answer the next question.\n\n{{#chunks}}- {{text}}\n{{/chunks...",
          "top_k": 50,
          "top_p": 0.8999999761581421
        }
      }
    }
  ],
  "errors": [],
  "interaction": {
    "interrupted": false,
    "components": [
      { "_type": "SlotComponent", "history_role": "none", "kind": "user",
        "name": "user_message", "producer": "user_message",
        "text": "How long have goldfish been kept by humans, and where were they first bred?" },
      { "_type": "KnowledgeComponent",
        "chunks[000].distance": "0.2349", "chunks[000].id": "goldfish_1",
        "chunks[000].text": "Goldfish (Carassius auratus) were selectively bred…",
        "chunks[001].distance": "0.3944", "chunks[001].id": "goldfish_4",
        "chunks[001].text": "Goldfish are cold-water fish that thrive at 65–72 °F…",
        "source": "retrieve" },
      { "_type": "SlotComponent", "history_role": "assistant", "kind": "general",
        "name": "generate", "producer": "generate",
        "text": "Goldfish have been kept by humans for over a thousand years…" }
    ]
  }
}
```

Reading it top to bottom: the turn succeeded in 452 ms with the first token at 125 ms; the retriever pulled 3 candidates, dropped 1 to the `0.4` threshold and attached 2; the generator received those chunks as a second system message before the user turn (`placement` `before_user_as_system`); and `tokens_reused: 0` against `tokens_decoded: 355` says the whole prompt was processed fresh — this was the agent's first turn.

Two things this payload does *not* show, because the turn did not do them: no `tool_calls` entries, and no `variable_replacements` on the Generate node.

______________________________________________________________________

## Related versioned shapes

Three schemas in Tryll carry their own version number. They move independently — do not read one as the other.

| Version field              | Belongs to                                  |
| -------------------------- | ------------------------------------------- |
| `schema_version`           | This payload, `TurnComplete.debug_info`.    |
| `format_version`           | QA and eval result files.                   |
| Monitor SSE schema version | The dev-time server monitor's event stream. |

______________________________________________________________________

## Related

- [Turn Inspector](https://docs.tryllengine.com/v0.8/reference/turn-inspector.html) — the editor panel that renders this payload
- [Agent Parameters](https://docs.tryllengine.com/v0.8/reference/agent-parameters.html#diagnostics) — `enable_diagnostics` and the rest of `CreateAgentRequest`
- [Server Configuration](https://docs.tryllengine.com/v0.8/reference/server-configuration.html) — `include_engine_diagnostics`, `include_interaction_in_diagnostics`
- [Node catalog](https://docs.tryllengine.com/v0.8/reference/nodes/index.html) — per-node diagnostics keys
- [How-to: Test an agent in the editor](https://docs.tryllengine.com/v0.8/how-to/test-agent-in-editor.html), [Use the Agent Log](https://docs.tryllengine.com/v0.8/how-to/use-agent-log.html), [Compare dialog variants](https://docs.tryllengine.com/v0.8/how-to/compare-dialog-variants.html)
- [Concepts: Projection and token budgets](https://docs.tryllengine.com/v0.8/concepts/projection-and-token-budgets.html)
