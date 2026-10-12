# TTS and Voice Output

Tryll supports server-side text-to-speech synthesis. Sherpa-ONNX hosts Supertonic 3 and Pocket TTS. llama.cpp runs Orpheus and Maya1, whose audio codes a shared SNAC 24 kHz ONNX decoder turns into sound on the CPU. See [TTS Models](https://docs.tryllengine.com/v0.8/reference/tts-models.html) for which catalog names are public and which controls each family accepts. Two graph nodes produce speech:

- [GenerateAndSpeak](https://docs.tryllengine.com/v0.8/reference/nodes/generate-and-speak.html) fuses LLM inference with TTS synthesis, streaming PCM audio frames alongside the text reply — lowest latency for a *generated* spoken answer.
- [Speak](https://docs.tryllengine.com/v0.8/reference/nodes/speak.html) is TTS-only: it voices the slot named by its `input` param with no LLM pass. The default is `user_message`, the player's own words, so set `input` to an upstream node's output (e.g. a [CannedResponse](https://docs.tryllengine.com/v0.8/reference/nodes/canned-response.html)).

Both stream the same PCM audio frames to the client and use the same voice parameters and playback components. They differ in whether generation is involved, and only `GenerateAndSpeak` can send a lip-sync viseme track.

## The streaming pipeline

When a `GenerateAndSpeak` node runs:

1. **Format announcement** — the node first sends one `TtsAudioFormatFrame`: sample rate, channels (1) and bits per sample (16). Every speaking node sends its own, so a turn with two speaking nodes sends two.
1. **Token generation** — the language model produces tokens, forwarded to the client as `AnswerText` deltas (identical to `Generate`).
1. **Sentence segmentation** — tokens accumulate until a sentence ends and the text since the last cut is at least `min_sentence_chars` wide (a CJK or other 3–4-byte character counts as 2):
   - `.` `!` `?` or `…` followed by a space or by the end of the text so far. A `.` right after a digit waits for the next character, so "3.14" stays whole.
   - At once: CJK `。！？｡`, Devanagari `। ॥`, Arabic `؟`, Urdu `۔`, Ethiopic `።` and `‼`. Closing quotes and brackets right after them (`」』）”`…) stay with the sentence.
   - With `first_chunk_max_chars` set, the *first* chunk may also end at `,` `;` `:` followed by a space or at an em dash, or at the next space once it is that wide — never before 5 words or `min_sentence_chars`. This has no effect on text without spaces, such as CJK.
   - Text that runs 16 KiB with no sentence end is cut at the last space, so the buffer stays bounded; the server logs a warning.
1. **TTS synthesis** — each chunk is synthesized to mono int16 PCM at the model's native sample rate and streamed as `TtsAudioFrame` messages. At most one chunk is synthesizing at a time: the next finished chunk waits for it, and the text stream pauses briefly while it waits.

This keeps time-to-first-audio low: the NPC starts speaking as soon as the first chunk is ready, while the language model is still generating the rest of the answer.

```
Client                          Server
  │                                │
  ├─ SendMessageRequest ──────────►│
  │◄── TtsAudioFormatFrame ────────┤  (node starts)
  │◄── AnswerText … ───────────────┤  (LLM tokens)
  │◄── TtsAudioFrame … ────────────┤  (first sentence)
  │◄── AnswerText … ───────────────┤  (LLM still running)
  │◄── TtsAudioFrame … ────────────┤  (next sentences)
  │◄── TurnComplete ───────────────┤  (after the last audio)
```

## Audio format

All TTS audio is **mono signed-16-bit PCM** at the sample rate reported in the `TtsAudioFormatFrame` frame. The sample rate is model-specific: for example 44 100 Hz for Supertonic 3 and 24 000 Hz for Orpheus and Maya1. Always read it from the format frame. The Unity speaker converts the int16 samples to float32; the Unreal speaker queues the int16 PCM directly.

## End of stream

`TtsAudioFrame` carries no `is_final` flag. The last frame arrives before `TurnComplete`. The audio queue on the client drains naturally — the playback component fills the audio buffer with silence on underrun until the queue is empty.

## Session configuration

There is no TTS engine to declare. Download a voice (a TTS model) in the Model Manager and name it in the node's `tts_model_name`. The catalog chooses the engine for each voice — Sherpa-ONNX for Supertonic / Pocket, llama.cpp for Orpheus / Maya1. A session warm-up hint is optional; see [Choose where models run](https://docs.tryllengine.com/v0.8/how-to/choose-where-models-run.html).

## Voice configuration

TTS voice settings live on the `GenerateAndSpeak` and `Speak` nodes (identical fields on both):

| Parameter                  | Structural | Runtime-mutable | Description                                                                                                                                                                                                                                                                                                        |
| -------------------------- | ---------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `tts_model_name`           | Yes        | No              | Catalog voice name. Optional — empty uses the first TTS voice in `models.json`.                                                                                                                                                                                                                                    |
| `speaker_id`               | No         | Yes             | Voice index, `0` to the model's speaker count − 1. Supertonic 3: `0`–`9`. Orpheus: `0`–`7` (Tara … Zoe). Maya1: `0`–`1` (or override with `tts_voice_description`). Pocket TTS is single-speaker: must stay `0` there — pick a Pocket voice with `tts_voice` instead. Out-of-range values fail agent creation.     |
| `speed`                    | No         | Yes             | Speech-rate multiplier. `1.0` = native speed. Supertonic 3 and Pocket TTS: `0.1 – 3.0`. Orpheus and Maya1: only `1.0`.                                                                                                                                                                                             |
| `tts_lang`                 | No         | Yes             | Synthesis language on multilingual models (Supertonic 3: 31 languages). Empty = the session locale, then the catalog default. If the voice does not support the session language, agent creation fails with `3017`.                                                                                                |
| `tts_voice`                | No         | Yes             | Reference-voice WAV (storage-root-relative) for voice-cloning models (Pocket TTS). Empty = model default voice. Rejected on every other family. The file must exist at agent creation. See [Clone a Voice](https://docs.tryllengine.com/v0.8/how-to/clone-a-voice.html).                                           |
| `tts_voice_description`    | No         | Yes             | Natural-language stable voice identity (age, accent, pitch, timbre). Empty = speaker preset / catalog default. Maya1 only — on another voice, agent creation fails. At most 1024 bytes, with no `"`, `<`, `>` or control characters.                                                                               |
| `tts_delivery_instruction` | No         | Yes             | Natural-language sustained delivery for this call (angry, whisper, …). Empty = neutral. Maya1 only, with the same limits.                                                                                                                                                                                          |
| `min_sentence_chars`       | No         | Yes             | Minimum width of a chunk before a sentence end closes it (a CJK or other 3–4-byte character counts as 2). Default `12`, two or three English words. Lower = shorter first-audio latency.                                                                                                                           |
| `first_chunk_max_chars`    | No         | Yes             | First chunk only: also end it at a clause mark, and at the next space once it is this wide (never under 5 words or `min_sentence_chars`). `0` = off. `48` cut the model-side wait before the first audio by about 40% in our measurements. Listen to it with your voice first: a clause can end on a falling tone. |

These are four different knobs. Do not encode identity or delivery as a spoken `[emotion]` prefix — that syntax is not supported. Model-native `<tag>` tokens (for example Maya `<laugh>`) may appear in the text the synthesizer hears; unknown tags are stripped from the audio copy only and do not rewrite history.

Structural parameters are locked at agent creation time. Mutable parameters can be changed between turns with `ChangeAgentParam` (see [Change Agent Parameters at Runtime](https://docs.tryllengine.com/v0.8/how-to/change-agent-parameters.html)). A bad value fails agent creation with `3003`, or a later change with `3007`.

If synthesis fails during a turn, the client gets an `ErrorResponse` with `4001` ("TTS synthesis failed") and the turn still completes; the text reply is not affected.

## Lip sync (visemes)

A `GenerateAndSpeak` node with `emit_visemes` on also sends a **viseme track**: the mouth shapes that go with the speech, and when each one starts. It uses the 15 standard Oculus visemes (`sil` for rest, `PP` for p/b/m, `aa` for an open vowel, …), which map onto a face rig's blendshapes.

The server builds the track for each sentence as soon as that sentence is synthesized, from two inputs:

- **the text** — the words are turned into speech sounds with an English pronunciation dictionary, and each sound into a mouth shape;
- **the synthesized audio** — silences become rests (anchored to the sentence's punctuation where possible), the shapes are spread over the spoken stretches in proportion to typical sound lengths, and louder speech opens the mouth further.

The track also keeps lip closures on screen long enough to read, merges shapes too short to see, and starts each shape slightly before its sound, as real lips do. It arrives as one `TtsVisemeFrame` per sentence, after that sentence's audio once its synthesis is done, so under normal buffering it reaches the client before that audio plays. A sentence that produced no audio sends no frame. Offsets are in samples from the start of this node's audio (its format announcement).

No extra model runs. What to expect:

- **Timing is estimated, not measured per sound.** Pauses, starts and ends follow the real audio closely; the shapes inside a stretch of speech are placed by proportion.
- **English pronunciation.** Text in other Latin-script languages gets approximate shapes from English spelling rules. Text in other scripts (Cyrillic, CJK, …) gets no shapes: the mouth stays at rest.
- **Voice-agnostic.** The track works with every TTS voice, since it only needs the text and the audio.

See [Add Lip Sync to a Character](https://docs.tryllengine.com/v0.8/how-to/add-lip-sync-to-a-character.html) for setup.

## Client-side playback components

The Tryll Unity and Unreal packages ship a ready-made playback component that wires the TTS audio stream to the platform audio engine:

| Platform | Component                                    | Audio backend                                     |
| -------- | -------------------------------------------- | ------------------------------------------------- |
| Unity    | `TryllSpeaker` (`MonoBehaviour`)             | Streaming `AudioClip` with `pcmReaderCallback`    |
| Unreal   | `UTryllSpeakerComponent` (`UActorComponent`) | `USoundWaveProcedural` + hidden `UAudioComponent` |

The Unreal speaker is always non-spatialized (2D). The Unity speaker plays through its `AudioSource` and leaves that source's spatial settings alone. For custom audio, such as your own spatializer, subscribe to the raw PCM: `TryllSpeaker.OnTtsPcm` (Unity) or `UTryllSpeakerComponent::OnTtsPcm` (Unreal). You can also replace the speaker entirely with `TryllAgent.SetOnTtsAudioFormat` / `SetOnTtsAudio`; each holds one callback, so setting it replaces the speaker's.

## See also

- [Add voice output to an agent](https://docs.tryllengine.com/v0.8/how-to/add-voice-to-an-agent.html) — step-by-step guide for Unity and Unreal
- [Filter LLM output artifacts](https://docs.tryllengine.com/v0.8/how-to/filter-llm-output-artifacts.html) — strip speaker prefixes and roleplay spans before TTS hears the stream
- [Substitute agent variables in LLM output](https://docs.tryllengine.com/v0.8/how-to/substitute-agent-variables-in-output.html) — replace `__PLACEHOLDER__` markers before TTS
- [Reference: TTS Models](https://docs.tryllengine.com/v0.8/reference/tts-models.html) — shipped voices (Sherpa-ONNX and llama.cpp SNAC), family matrix, and licenses
- [Reference: GenerateAndSpeak node](https://docs.tryllengine.com/v0.8/reference/nodes/generate-and-speak.html)
- [Reference: Speak node](https://docs.tryllengine.com/v0.8/reference/nodes/speak.html)
- [Reference: TryllSpeaker (Unity)](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-speaker.html)
- [Add lip sync to a character](https://docs.tryllengine.com/v0.8/how-to/add-lip-sync-to-a-character.html) — play the viseme track on a face
- [Reference: TryllLipSync (Unity)](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-lip-sync.html)
- [Use voice input](https://docs.tryllengine.com/v0.8/how-to/use-voice-input.html) — pair with TTS for full two-way voice interaction
