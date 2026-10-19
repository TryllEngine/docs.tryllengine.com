# Speak

The Speak node is a TTS-only step: it voices whatever text its `input` selector resolves to, segments it into sentences, dispatches each sentence to a dedicated [TTS](https://docs.tryllengine.com/v0.9/reference/glossary.html#tts) strand, and emits PCM audio frames — without running any [language-model](https://docs.tryllengine.com/v0.9/reference/glossary.html#language-model) inference. Use it to voice upstream-produced text, for example `CannedResponse → Speak` so a scripted line is spoken without paying for a generation pass.

The text it speaks is selected by `input`, exactly like every other input-bearing node: leave it empty to voice the `user_message` slot, or set it to the name of any upstream node's output slot (e.g. a preceding [Generate](https://docs.tryllengine.com/v0.9/reference/nodes/generate.html) or [CannedResponse](https://docs.tryllengine.com/v0.9/reference/nodes/canned-response.html)) to voice that instead. If the resolved slot is missing or empty, the node synthesizes nothing and exits via `default`.

Speak only sees text that an upstream node has already finished producing, so it cannot overlap synthesis with generation. Chained after `Generate`, audio begins only once generation has fully completed (Speak then pipelines TTS across the sentences of that completed text). When the goal is simply to speak a generated answer with the lowest possible delay, use [Generate and Speak](https://docs.tryllengine.com/v0.9/reference/nodes/generate-and-speak.html), which streams synthesis *while* the LLM is still generating. Reach for a standalone Speak when the text it voices did not come from a generation pass (e.g. `CannedResponse → Speak`), or when you need to branch on / transform the text before voicing it.

The node has no voice settings. It speaks with the agent's [voice profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html), which you pass when you create the agent. A graph with a Speak node and no voice fails agent creation with `3025 AgentVoiceRequired`. The model, speaker, speed, language, reference clip and delivery mode all come from the profile. The node keeps only its `input` and its chunking (`min_sentence_chars`, `first_chunk_max_chars`).

The audio format is announced once per turn (via `OnTtsAudioFormat`), by the first speaking node that produces audio. If an earlier node of the same turn already spoke, Speak reuses that format. End-of-stream is signalled by turn completion. Unlike [Generate and Speak](https://docs.tryllengine.com/v0.9/reference/nodes/generate-and-speak.html), Speak does not emit answer-text deltas: the upstream node that produced the text already did. When the voice has `emit_visemes` on, Speak also sends a lip-sync viseme track.

`NodeType`: `Speak`.

## Parameters

| Param                   | Type | Default | Range | Structural | Localizable | Description                                                                                                                                                                                                                                                                                                                                                                                                            |
| ----------------------- | ---- | ------- | ----- | ---------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `input`                 | str  | empty   | —     | ✓          | —           | Slot name this node voices. Empty = "user_message" (same default as every other input-bearing node). Structural: immutable after creation — rebinding would re-wire the slot dataflow that is validated once at agent creation.                                                                                                                                                                                        |
| `min_sentence_chars`    | int  | 12      | ≥ 1.0 | —          | —           | Minimum width of the text since the last cut before a sentence end closes a chunk; a CJK or other 3–4-byte character counts as 2. The default 12 is two or three English words. Lower values reduce first-utterance latency at the cost of more, smaller TTS calls.                                                                                                                                                    |
| `first_chunk_max_chars` | int  | 0       | ≥ 0.0 | —          | —           | Time-to-first-audio knob. When > 0, the FIRST chunk may also end at a comma, semicolon or colon followed by a space, or at an em dash, and is cut at the next space once it is this wide, so speech starts sooner. Neither cut fires before 5 words or min_sentence_chars. Later chunks keep whole sentences. No effect on text without spaces (CJK). 0 = off. Check how it sounds with your voice before shipping it. |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit      | Param field    | Description                                                                      |
| --------- | -------------- | -------------------------------------------------------------------------------- |
| `default` | `default_exit` | Default exit target — routes here after synthesis completes. Empty string = END. |

## Exit routes

| Route     | Fires when                                                                      |
| --------- | ------------------------------------------------------------------------------- |
| `default` | Always, after synthesis completes (including when the resolved text was empty). |

## Side effects

- Emits `OnTtsAudioFormat` once per turn (unless the resolved text is empty, or an earlier speaking node of the turn already announced it), then `OnTtsAudio` PCM frames as sentences are synthesized.
- With the voice's `emit_visemes` on, emits one `TtsVisemeFrame` per synthesized sentence. Offsets count from the turn's format announcement.
- Does not write a slot or emit `OnAnswerText` (Speak only voices an existing slot; it produces no text of its own).

## Diagnostics

When `enable_diagnostics = true`, this node contributes to `debug_info.nodes[].diagnostics`. See [Turn Diagnostics JSON](https://docs.tryllengine.com/v0.9/reference/turn-diagnostics-json.html) for the envelope.

Speak runs no language model, so it emits no `input`, `output` or `engine` section. It emits a `parameters` section for its own settings and a `voice` section for the agent's voice as it was when the node ran.

| Key                                 | Meaning                                                                                                                                                                                                                                                                                           | Absent when                       |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| `parameters.input`                  | The slot whose text was voiced.                                                                                                                                                                                                                                                                   | Using the `user_message` default. |
| `voice.*`                           | The agent's voice: `tts_model_name`, `speaker_id`, `speed`, `lang`, `reference_voice`, `voice_description`, `delivery_mode`, `intensity`, `delivery_instruction`, `emit_visemes`, `sample_rate`. See [Voice Profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html#diagnostics). | Per key, as listed there.         |
| `parameters.min_sentence_chars`     | Sentence-boundary threshold before a chunk is synthesized.                                                                                                                                                                                                                                        | —                                 |
| `parameters.first_chunk_max_chars`  | First-chunk length cap (`0` = off).                                                                                                                                                                                                                                                               | —                                 |
| `parameters.time_to_first_audio_ms` | Delay before the first audio chunk was produced.                                                                                                                                                                                                                                                  | No audio was produced.            |

Because the node produces no text, an unexpectedly silent turn is diagnosed upstream: check the `output.text` of whichever node fed `input`, and the `interaction` slot snapshot. If the text was there, look for an `ErrorResponse` with `4001` ("TTS synthesis failed"): the turn still completes when synthesis throws.

## Related

- [Voice Profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html) — the agent's voice that this node speaks with.
- [Generate and Speak](https://docs.tryllengine.com/v0.9/reference/nodes/generate-and-speak.html) — fused LLM generation + TTS.
- [Canned Response](https://docs.tryllengine.com/v0.9/reference/nodes/canned-response.html) — common upstream producer of the assistant text Speak voices.
- [Concept: Slots and inter-node value passing](https://docs.tryllengine.com/v0.9/concepts/slots-and-inter-node-value-passing.html) — how `input` resolution works.
