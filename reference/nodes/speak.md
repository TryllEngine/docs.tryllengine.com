# Speak

The Speak node is a TTS-only step: it voices whatever text its `input` selector resolves to, segments it into sentences, dispatches each sentence to a dedicated [TTS](https://docs.tryllengine.com/reference/glossary.html#tts) strand, and emits PCM audio frames — without running any [language-model](https://docs.tryllengine.com/reference/glossary.html#language-model) inference. Use it to voice upstream-produced text, for example `CannedResponse → Speak` so a scripted line is spoken without paying for a generation pass.

The text it speaks is selected by `input`, exactly like every other input-bearing node: leave it empty to voice the `user_message` slot, or set it to the name of any upstream node's output slot (e.g. a preceding [Generate](https://docs.tryllengine.com/reference/nodes/generate.html) or [CannedResponse](https://docs.tryllengine.com/reference/nodes/canned-response.html)) to voice that instead. If the resolved slot is missing or empty, the node synthesizes nothing and exits via `default`.

Speak only sees text that an upstream node has already finished producing, so it cannot overlap synthesis with generation. Chained after `Generate`, audio begins only once generation has fully completed (Speak then pipelines TTS across the sentences of that completed text). When the goal is simply to speak a generated answer with the lowest possible delay, use [Generate and Speak](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html), which streams synthesis *while* the LLM is still generating. Reach for a standalone Speak when the text it voices did not come from a generation pass (e.g. `CannedResponse → Speak`), or when you need to branch on / transform the text before voicing it.

The voice is chosen by `tts_model_name`. Leave it empty to use the first TTS voice available in the server's model catalog, or set it to pick a specific voice.

Audio format is announced once (via `OnTtsAudioFormat`) before the first audio chunk; end-of-stream is signalled by turn completion. Unlike [Generate and Speak](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html), Speak does not emit answer-text deltas — the upstream node that produced the text already did.

`NodeType`: `Speak`.

## Parameters

| Param                      | Type                      | Default                         | Range     | Structural | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| -------------------------- | ------------------------- | ------------------------------- | --------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `tts_model_name`           | Optional[str]             | inherit model default           | —         | ✓          | TTS model catalog name. Optional — when empty the server uses the first available TTS voice in the model catalog. Set it to pick a specific voice.                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `speaker_id`               | int                       | 0                               | ≥ 0.0     | —          | TTS speaker / voice index (default 0). Valid range is 0 to the model's speaker count minus 1 — the shipped Supertonic 3 bundle has 10 voice styles (0–9); Pocket TTS is single-speaker, so it must stay 0 there (pick a Pocket voice with tts_voice instead). Agent creation fails with an out-of-range error for values the model cannot satisfy.                                                                                                                                                                                                                                                                 |
| `speed`                    | float                     | 1.0                             | 0.1 – 3.0 | —          | TTS speech-rate multiplier. 1.0 = native speed.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `tts_lang`                 | Optional[str]             | session locale -> model default | —         | —          | TTS language hint for multilingual models that select the language per call (e.g. Supertonic 3: "en", "de", "fr", …). Resolution order: this param, then the session locale (CreateSessionRequest.locale), then the model's catalog default. Set it only to override the session — one character who keeps speaking French in a German playthrough. For a whole-game language switch, set the session locale instead and leave this empty: it reaches every agent and every node with no per-node mutation. Ignored by families whose language is fixed by the model itself (e.g. Pocket TTS per-language models). |
| `tts_voice`                | Optional[str]             | inherit model default           | —         | —          | Reference-voice WAV for zero-shot voice-cloning families (Pocket TTS): a path relative to the session storage root pointing at a 10–20 s clean mono recording of the target voice. Empty = the model's default voice. Ignored by fixed-voice families (use speaker_id there).                                                                                                                                                                                                                                                                                                                                      |
| `input`                    | Optional[str]             | inherit model default           | —         | ✓          | Slot name this node voices. Empty = "user_message" (same default as every other input-bearing node). Structural: immutable after creation — rebinding would re-wire the slot dataflow that is validated once at agent creation.                                                                                                                                                                                                                                                                                                                                                                                    |
| `min_sentence_chars`       | int                       | 12                              | ≥ 1.0     | —          | Minimum characters before a sentence boundary fires. Lower values reduce first-utterance latency at the cost of more, smaller TTS calls.                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `tts_voice_description`    | Optional[str] (multiline) | inherit model default           | —         | —          | Natural-language stable voice identity: age, accent, pitch, timbre and baseline pacing. Empty = use speaker_id / catalog default. Appended after default_exit so existing FlatBuffers field IDs do not move.                                                                                                                                                                                                                                                                                                                                                                                                       |
| `tts_delivery_instruction` | Optional[str] (multiline) | inherit model default           | —         | —          | Natural-language sustained delivery direction for this synthesis call. Empty = neutral / model default.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |

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

- Emits `OnTtsAudioFormat` once, then `OnTtsAudio` PCM frames as sentences are synthesized.
- Does not write a slot or emit `OnAnswerText` (Speak only voices an existing slot; it produces no text of its own).

## Diagnostics

When `enable_diagnostics = true`, this node contributes to `debug_info.nodes[].diagnostics`. See [Turn Diagnostics JSON](https://docs.tryllengine.com/reference/turn-diagnostics-json.html) for the envelope.

Speak runs no language model, so it emits **only** a `parameters` section — no `input`, `output` or `engine`. Unlike [Generate and Speak](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html#diagnostics), the voice settings live in `parameters` rather than in a separate `tts` section.

| Key                                   | Meaning                                                    | Absent when                       |
| ------------------------------------- | ---------------------------------------------------------- | --------------------------------- |
| `parameters.input`                    | The slot whose text was voiced.                            | Using the `user_message` default. |
| `parameters.speaker_id`               | Speaker index within the voice model.                      | —                                 |
| `parameters.speed`                    | Playback rate multiplier.                                  | —                                 |
| `parameters.tts_lang`                 | Requested synthesis language.                              | —                                 |
| `parameters.tts_voice`                | Reference voice used.                                      | —                                 |
| `parameters.tts_voice_description`    | The voice description prompt.                              | Not set.                          |
| `parameters.tts_delivery_instruction` | The delivery instruction.                                  | Not set.                          |
| `parameters.sample_rate`              | The TTS model's native sample rate, in Hz.                 | —                                 |
| `parameters.min_sentence_chars`       | Sentence-boundary threshold before a chunk is synthesized. | —                                 |
| `parameters.time_to_first_audio_ms`   | Delay before the first audio chunk was produced.           | No audio was produced.            |

Because the node produces no text, an unexpectedly silent turn is diagnosed upstream: check the `output.text` of whichever node fed `input`, and the `interaction` slot snapshot.

## Related

- [Generate and Speak](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html) — fused LLM generation + TTS.
- [Canned Response](https://docs.tryllengine.com/reference/nodes/canned-response.html) — common upstream producer of the assistant text Speak voices.
- [Concept: Slots and inter-node value passing](https://docs.tryllengine.com/concepts/slots-and-inter-node-value-passing.html) — how `input` resolution works.
