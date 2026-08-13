# TTS and Voice Output

Tryll supports server-side text-to-speech synthesis using the [Sherpa-ONNX](https://github.com/k2-fsa/sherpa-onnx) engine. Two graph nodes produce speech:

- [GenerateAndSpeak](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html) fuses LLM inference with TTS synthesis, streaming PCM audio frames alongside the text reply — lowest latency for a *generated* spoken answer.
- [Speak](https://docs.tryllengine.com/reference/nodes/speak.html) is TTS-only: it voices the latest assistant text produced upstream (e.g. a [CannedResponse](https://docs.tryllengine.com/reference/nodes/canned-response.html)) with no LLM pass.

Both stream the same PCM audio frames to the client and use the same voice parameters and playback components; they differ only in whether generation is involved.

## The streaming pipeline

When a turn starts on a `GenerateAndSpeak` node, three concurrent things happen:

1. **Token generation** — the language model produces tokens, forwarded to the client as `AnswerText` deltas (identical to `Generate`).
1. **Sentence segmentation** — tokens accumulate until a sentence boundary is detected or `min_sentence_chars` is reached. Each completed sentence is dispatched to the TTS strand immediately — no waiting for the full response.
1. **TTS synthesis** — the TTS model synthesizes the sentence to mono int16 PCM at the model's native sample rate, then streams the audio back as a sequence of `TtsAudioFrame` wire messages, preceded by a single `TtsAudioFormatFrame` message that declares the sample rate.

This pipeline keeps time-to-first-audio low: the NPC starts speaking as soon as the first sentence is complete, while the language model is still generating the rest of the answer.

```
Client                      Server
  │                            │
  ├─ SendMessage ─────────────►│
  │                            ├─ LLM tokens → AnswerText chunks ──────────►│
  │                            ├─ Sentence ready → TTS synthesis
  │◄── TtsAudioFormatFrame ────┤  (first sentence)
  │◄── TtsAudioFrame … ────────┤
  │◄── AnswerText … ───────────┤  (LLM still running)
  │◄── TtsAudioFrame … ────────┤  (second sentence)
  │◄── AnswerText … ───────────┤
  │◄── TurnComplete ───────────┤
```

## Audio format

All TTS audio is **mono signed-16-bit PCM** at the sample rate reported in the `TtsAudioFormatFrame` frame. The sample rate is model-specific (commonly 22 050 Hz or 24 000 Hz). The client's speaker component converts int16 samples to float32 before passing them to the audio engine.

## End of stream

`TtsAudioFrame` carries no `is_final` flag. The last frame arrives before `TurnComplete`. The audio queue on the client drains naturally — the playback component fills the audio buffer with silence on underrun until the queue is empty.

## Session configuration

You must declare a TTS engine when you create the session. Without one, a `GenerateAndSpeak` node produces text only and audio frames are not emitted.

```
// Unity
client.CreateSession(
    TryllInferenceEngine.LlamaCpp,
    ttsEngine: TryllInferenceEngine.SherpaOnnx);
```

```
// Unreal (C++)
Subsystem->CreateSession(
    ETryllInferenceEngine::LlamaCpp,
    /*GameName=*/ TEXT(""),
    /*SttEngine=*/ ETryllInferenceEngine::Mock,
    /*TtsEngine=*/ ETryllInferenceEngine::SherpaOnnx);
```

## Voice configuration

TTS voice settings live on the `GenerateAndSpeak` and `Speak` nodes (identical fields on both):

| Parameter            | Structural | Runtime-mutable | Description                                                                                                                                                                                                                                                  |
| -------------------- | ---------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `tts_model_name`     | Yes        | No              | Catalog voice name. Optional — empty uses the first TTS voice in `models.json`.                                                                                                                                                                              |
| `speaker_id`         | No         | Yes             | Voice index, `0` to the model's speaker count − 1 (the shipped Supertonic 3 bundle has 10 voice styles, `0`–`9`). Pocket TTS is single-speaker: must stay `0` there — pick a Pocket voice with `tts_voice` instead. Out-of-range values fail agent creation. |
| `speed`              | No         | Yes             | Speech-rate multiplier. `1.0` = native speed, range `0.1 – 3.0`.                                                                                                                                                                                             |
| `tts_lang`           | No         | Yes             | Synthesis language on multilingual models (Supertonic 3: 31 languages). Empty = catalog default.                                                                                                                                                             |
| `tts_voice`          | No         | Yes             | Reference-voice WAV (storage-root-relative) for voice-cloning models (Pocket TTS). Empty = model default voice. See [Clone a Voice](https://docs.tryllengine.com/how-to/clone-a-voice.html).                                                                 |
| `min_sentence_chars` | No         | Yes             | Minimum characters before a sentence fires to TTS. Lower = shorter first-audio latency.                                                                                                                                                                      |

Structural parameters are locked at agent creation time. Mutable parameters can be changed between turns with `ChangeAgentParam` (see [Change Agent Parameters at Runtime](https://docs.tryllengine.com/how-to/change-agent-parameters.html)).

## Client-side playback components

The Tryll Unity and Unreal packages ship a ready-made playback component that wires the TTS audio stream to the platform audio engine:

| Platform | Component                                    | Audio backend                                     |
| -------- | -------------------------------------------- | ------------------------------------------------- |
| Unity    | `TryllSpeaker` (`MonoBehaviour`)             | Streaming `AudioClip` with `pcmReaderCallback`    |
| Unreal   | `UTryllSpeakerComponent` (`UActorComponent`) | `USoundWaveProcedural` + hidden `UAudioComponent` |

Both are non-spatialized (2D) by default, suitable for NPC narration. For spatial audio, subscribe to `TryllAgent.OnTtsAudioFormat` and `OnTtsAudio` directly and feed the raw PCM chunks to a custom spatializer.

## See also

- [Add voice output to an agent](https://docs.tryllengine.com/how-to/add-voice-to-an-agent.html) — step-by-step guide for Unity and Unreal
- [Filter LLM output artifacts](https://docs.tryllengine.com/how-to/filter-llm-output-artifacts.html) — strip speaker prefixes and roleplay spans before TTS hears the stream
- [Substitute agent variables in LLM output](https://docs.tryllengine.com/how-to/substitute-agent-variables-in-output.html) — replace `__PLACEHOLDER__` markers before TTS
- [Reference: TTS Models](https://docs.tryllengine.com/reference/tts-models.html) — which Sherpa-ONNX voice families are supported, which are not, and why
- [Reference: GenerateAndSpeak node](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html)
- [Reference: Speak node](https://docs.tryllengine.com/reference/nodes/speak.html)
- [Reference: TryllSpeaker (Unity)](https://docs.tryllengine.com/reference/client-api/unity/tryll-speaker.html)
- [Use voice input](https://docs.tryllengine.com/how-to/use-voice-input.html) — pair with TTS for full two-way voice interaction
