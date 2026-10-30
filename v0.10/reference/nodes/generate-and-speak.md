# Generate and Speak

The Generate-and-Speak node fuses LLM token generation with streaming [TTS](https://docs.tryllengine.com/v0.10/reference/glossary.html#tts) synthesis: it runs the same projection and [language-model](https://docs.tryllengine.com/v0.10/reference/glossary.html#language-model) inference as a [Generate](https://docs.tryllengine.com/v0.10/reference/nodes/generate.html) node, but as tokens arrive it segments them into sentences, dispatches each sentence to a dedicated TTS strand, and emits PCM audio frames alongside the text deltas.

Because synthesis overlaps generation, audio begins after roughly the first sentence (`min_sentence_chars`) rather than after the whole answer — the TTS of one sentence runs while the LLM is still generating the next. This is the lowest-latency way to speak a generated answer (measured by the `time_to_first_audio_ms` diagnostic). `sampling.max_tokens` is fixed at creation, the same as on [Generate](https://docs.tryllengine.com/v0.10/reference/nodes/generate.html). Chaining a standalone [Generate](https://docs.tryllengine.com/v0.10/reference/nodes/generate.html) → [Speak](https://docs.tryllengine.com/v0.10/reference/nodes/speak.html) instead (with Speak's `input` set to the Generate node's output) produces the same voice but cannot start until generation has fully finished, since `Speak` only sees the completed text. Prefer this fused node when you want a generated answer spoken with minimal delay, and the split form only when you need to branch on, inspect, or transform the text in between.

The node has no voice settings. It speaks with the agent's [voice profile](https://docs.tryllengine.com/v0.10/reference/voice-profile.html), which you pass when you create the agent. A graph with a Generate and Speak node and no voice fails agent creation with `3025 AgentVoiceRequired`. The TTS model, speaker, speed, language, reference clip, delivery mode and `emit_visemes` all come from the profile. The node keeps its language-model settings and its chunking (`min_sentence_chars`, `first_chunk_max_chars`).

The audio format is announced once per turn, by the first speaking node that produces audio, so a later [Speak](https://docs.tryllengine.com/v0.10/reference/nodes/speak.html) node in the same turn reuses it. With the voice's `emit_visemes` on, the node sends one lip-sync viseme frame per sentence.

`NodeType`: `GenerateAndSpeak`.

Leave `context_size` at `0` to inherit the model variant's window, then the server `default_n_ctx` (commonly 8192) — the same resolution as [Generate](https://docs.tryllengine.com/v0.10/reference/nodes/generate.html). The `context` table is an advanced structural override. Do not set `q4_0` KV expecting a VRAM win at a right-sized window.

## Parameters

### Input

| Param   | Type | Default | Range | Structural | Localizable | Description                                                                                             |
| ------- | ---- | ------- | ----- | ---------- | ----------- | ------------------------------------------------------------------------------------------------------- |
| `input` | str  | empty   | —     | ✓          | —           | Text this node answers: the player's message or an earlier node's output. Empty = the player's message. |

### Model

| Param          | Type | Default | Range | Structural | Localizable | Description                                                                                                                       |
| -------------- | ---- | ------- | ----- | ---------- | ----------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `model_name`   | str  | empty   | —     | ✓          | —           | Language model this node runs. Empty = the workflow's Default Model Name.                                                         |
| `context_size` | int  | 0       | ≥ 0.0 | ✓          | —           | Context window for this node, in tokens; must fit the model's trained maximum. 0 = the model's own size, else the server default. |

### Prompt

| Param           | Type            | Default                      | Range | Structural | Localizable | Description                                                                                                                         |
| --------------- | --------------- | ---------------------------- | ----- | ---------- | ----------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `system_prompt` | str (multiline) | empty                        | —     | —          | ✓           | System message the model reads before the dialog. Empty = none.                                                                     |
| `template`      | str (multiline) | empty                        | —     | —          | ✓           | Mustache text added next to the Input (where: see Placement), for example the slot. tag or retrieved knowledge. Empty = none.       |
| `placement`     | Placement       | Placement.BeforeUserAsSystem | —     | —          | —           | Where Template goes: before or after the Input, as a system or a user message.                                                      |
| `grammar`       | str (multiline) | empty                        | —     | —          | —           | GBNF grammar the spoken answer must follow; needs a `root` rule. Empty = free text. Use it to limit speech to a fixed set of lines. |

### Behavior

| Param                   | Type | Default | Range | Structural | Localizable | Description                                                                                                                                 |
| ----------------------- | ---- | ------- | ----- | ---------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `min_sentence_chars`    | int  | 12      | ≥ 1.0 | —          | —           | Minimum length of a spoken chunk before a sentence end cuts it (CJK characters count double). Lower = speech starts sooner, more TTS calls. |
| `first_chunk_max_chars` | int  | 0       | ≥ 0.0 | —          | —           | Lets the first chunk end early, at a comma, dash or space once it is this long, so speech starts sooner. 0 = off; try 48.                   |

### Output

| Param                        | Type                  | Default               | Range | Structural | Localizable | Description                                                                                                                   |
| ---------------------------- | --------------------- | --------------------- | ----- | ---------- | ----------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `output_name`                | str                   | empty                 | —     | ✓          | —           | Name later nodes use to read this node's answer (in their Input, or as the slot. tag). Empty = this node's name.              |
| `history_role`               | HistoryRole           | HistoryRole.Assistant | —     | —          | —           | Whether later turns remember this answer as an assistant message. None = forgotten after this turn.                           |
| `substitute_agent_variables` | bool                  | False                 | —     | —          | —           | Replace **NAME** markers in the answer with the values of agent variables that allow output substitution. Not case-sensitive. |
| `output_filter`              | OutputFilterOverrides | unset                 | —     | —          | —           | Removes roleplay artifacts before they are spoken: "Bob:" prefixes, *actions*, (asides), [notes]. All off by default.         |

### Advanced

| Param        | Type              | Default | Range | Structural | Localizable | Description                                                                                                             |
| ------------ | ----------------- | ------- | ----- | ---------- | ----------- | ----------------------------------------------------------------------------------------------------------------------- |
| `sampling`   | SamplingOverrides | unset   | —     | —          | —           | Overrides for the model's sampling settings (temperature, top-p, max tokens…). Unset values keep the model's own.       |
| `context`    | ContextOverrides  | unset   | —     | ✓          | —           | Advanced memory and batching settings for this node's model (KV cache type, GPU offload, batch sizes). Unset = inherit. |
| `adaptation` | ModelAdaptation   | unset   | —     | ✓          | —           | LoRA adapters applied to this node's model. Empty = the base model.                                                     |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit      | Param field    | Description                                                                 |
| --------- | -------------- | --------------------------------------------------------------------------- |
| `default` | `default_exit` | Node to run after the answer is generated and spoken. Empty = end the turn. |

LoRA adapters

`adaptation.loras` runs this node's model with one or more fine-tuned LoRA adapters, each with a scale. See [Use a LoRA Adapter](https://docs.tryllengine.com/v0.10/how-to/use-a-lora-adapter.html) for where to put the file and how to tune the scale.

## Constrained output (GBNF grammar)

Like [Generate](https://docs.tryllengine.com/v0.10/reference/nodes/generate.html#constrained-output-gbnf-grammar), this node accepts an optional `grammar` (GBNF) that constrains the generated output at every decode step. The full rules there apply verbatim (must contain a `root` rule; fail-fast at agent creation and `InvalidParamValue` 3007 on a bad mutation; mutable per turn; constrain the wire, not the thinking).

The fit here is **narrower** than on Generate: the constrained output is also *spoken*, so grammar only makes sense when the constrained text is itself meant to be voiced — e.g. restricting an NPC to a fixed set of barks or canned lines:

```
root ::= "Halt!" | "Who goes there?" | "Move along." | "For the throne!"
```

Note the interaction with `min_sentence_chars` (default 12): very short constrained output may never cross the sentence-boundary threshold, so the whole (tiny) output flushes as a single TTS call — which is fine for barks.

See [Concept: Constrained output (GBNF)](https://docs.tryllengine.com/v0.10/concepts/constrained-output.html) for what a grammar does and doesn't guarantee, and [How to constrain output with a grammar](https://docs.tryllengine.com/v0.10/how-to/constrain-output-with-a-grammar.html) for a worked example.

## Diagnostics

When `enable_diagnostics = true`, this node contributes to `debug_info.nodes[].diagnostics`. See [Turn Diagnostics JSON](https://docs.tryllengine.com/v0.10/reference/turn-diagnostics-json.html) for the envelope, the shared `input.prompt[]` shape, and the `engine` counters.

This is the only node that emits a fifth section, **`tts`** — the voice half of the fused step is kept separate from the language-model parameters. It holds the agent's [voice](https://docs.tryllengine.com/v0.10/reference/voice-profile.html#diagnostics) as it was when the node ran, plus this node's chunking.

| Key                                                                                                                                                                                            | Meaning                                                                                                                                                                             | Absent when                                        |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| `parameters.model_name`                                                                                                                                                                        | The language model that actually ran, after fallback.                                                                                                                               | The name could not be resolved.                    |
| `parameters.temperature`, `top_p`, `top_k`, `max_tokens`                                                                                                                                       | The resolved sampling set. Unlike [Generate](https://docs.tryllengine.com/v0.10/reference/nodes/generate.html), the penalty and `seed` keys are not recorded.                       | —                                                  |
| `parameters.history_role`                                                                                                                                                                      | How the answer replays into later turns.                                                                                                                                            | —                                                  |
| `parameters.substitute_agent_variables`                                                                                                                                                        | `true`.                                                                                                                                                                             | Output substitution is off.                        |
| `parameters.output_filter.strip_speaker_prefix`, `parameters.output_filter.strip_asterisk_spans`, `parameters.output_filter.strip_paren_spans`, `parameters.output_filter.strip_bracket_spans` | `true`, one key per enabled rule.                                                                                                                                                   | That rule is off.                                  |
| `parameters.output_filter.max_prefix_bytes`, `parameters.output_filter.max_span_bytes`                                                                                                         | The scan limits the filter ran with.                                                                                                                                                | No filter rule is enabled.                         |
| `parameters.input`                                                                                                                                                                             | The configured input slot name.                                                                                                                                                     | Using the `user_message` default.                  |
| `parameters.output_name`                                                                                                                                                                       | The configured output slot name.                                                                                                                                                    | Defaulting to the node name.                       |
| `tts.tts_model_name`, `speaker_id`, `speed`, `lang`, `reference_voice`                                                                                                                         | The agent's voice as authored. An empty `lang` means the session locale or catalog default applied; `reference_voice` is the authored path, not a per-locale file resolved from it. | —                                                  |
| `tts.voice_description`                                                                                                                                                                        | The Maya1 voice description.                                                                                                                                                        | Not set.                                           |
| `tts.delivery_mode`, `tts.intensity`                                                                                                                                                           | The active delivery mode (`neutral` when none) and its intensity.                                                                                                                   | —                                                  |
| `tts.delivery_instruction`                                                                                                                                                                     | The active mode's instruction.                                                                                                                                                      | The mode has none.                                 |
| `tts.emit_visemes`                                                                                                                                                                             | Whether the voice sends a viseme track.                                                                                                                                             | —                                                  |
| `tts.sample_rate`                                                                                                                                                                              | The TTS model's native sample rate, in Hz.                                                                                                                                          | —                                                  |
| `tts.min_sentence_chars`                                                                                                                                                                       | Sentence-boundary threshold before a chunk is synthesized.                                                                                                                          | —                                                  |
| `tts.first_chunk_max_chars`                                                                                                                                                                    | First-chunk length cap (`0` = off).                                                                                                                                                 | —                                                  |
| `tts.time_to_first_audio_ms`                                                                                                                                                                   | Delay before the first audio chunk was produced — the voice counterpart of `time_to_first_token_ms`.                                                                                | No audio was produced.                             |
| `input.prompt[]`                                                                                                                                                                               | The rendered prompt as the model received it.                                                                                                                                       | —                                                  |
| `output.text`                                                                                                                                                                                  | The text this node produced (and spoke).                                                                                                                                            | The node produced no output slot.                  |
| `engine.*`                                                                                                                                                                                     | Grammar state and scheduler counters.                                                                                                                                               | Server config `include_engine_diagnostics` is off. |

The node-level `variable_replacements[]` and `output_filter_removals[]` arrays behave exactly as on [Generate](https://docs.tryllengine.com/v0.10/reference/nodes/generate.html#diagnostics), and matter more here: what the filter removed is also what the voice did *not* say.

## Related

- [How-to: Use a LoRA adapter](https://docs.tryllengine.com/v0.10/how-to/use-a-lora-adapter.html)
- [How to filter LLM output artifacts](https://docs.tryllengine.com/v0.10/how-to/filter-llm-output-artifacts.html)
- [How to substitute agent variables in LLM output](https://docs.tryllengine.com/v0.10/how-to/substitute-agent-variables-in-output.html)
- [Concept: TTS and voice output](https://docs.tryllengine.com/v0.10/concepts/tts-and-voice-output.html)
- [Voice Profile](https://docs.tryllengine.com/v0.10/reference/voice-profile.html) — the agent's voice that this node speaks with
- [Generate](https://docs.tryllengine.com/v0.10/reference/nodes/generate.html)
