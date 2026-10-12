# STT and Voice Input

Voice input lets a player **speak** to an agent instead of typing. The client streams raw microphone audio to the server, a **speech-to-text (STT)** model transcribes it, and the transcript comes back as a stream of updates. The client can then send the final text to an agent as if the player had typed it.

This page explains the mental model: the utterance lifecycle, the two choices that shape how a turn ends, and the kinds of transcript update you receive. For the step-by-step API, see [Use Voice Input](https://docs.tryllengine.com/v0.8/how-to/use-voice-input.html); to bias recognition toward game-specific words, see [Bias Voice Input with Hotwords](https://docs.tryllengine.com/v0.8/how-to/use-voice-input-hotwords.html).

## The pipeline

```
flowchart LR
    mic["Microphone<br>(client)"] -->|PCM frames| handle["VoiceInput handle"]
    handle -->|AudioBuffer| server["Server STT session<br>(Sherpa-ONNX)"]
    server -->|TranscriptUpdate| handle
    handle -.->|UtteranceFinal, then SendMessage| agent["Agent (optional)"]
```

You create a **VoiceInput handle** (bound to an STT model), open an **utterance**, stream 16-bit PCM audio as it's captured, and receive **transcript updates**. The server resamples your audio to the model's rate, runs it through the STT engine, and pushes updates back. A non-empty final transcript is text the client can send to an agent; the server does not start that turn.

Voice input needs an **STT model** on disk — download one in the Model Manager. Its catalog entry names its engine (Sherpa-ONNX), so the session needs no engine setting. STT runs on the **CPU**, so it costs system RAM but **no VRAM**; see [Models and Inference Engines](https://docs.tryllengine.com/v0.8/concepts/models-and-inference-engines.html).

## The utterance lifecycle

An **utterance** is one `Begin … End` cycle on a handle — a single thing the player says.

1. **Begin** — opens a recording window and starts mic capture. The server only transcribes. On a non-empty final, send the text to an agent yourself (the Unity and Unreal voice components do this when a target agent is set, and drop it when that agent already has a turn in flight).
1. **Stream audio** — push PCM chunks as they arrive. Sending is fire-and-forget; chunks that arrive with no open utterance are dropped.
1. **Updates arrive** — the server streams [transcript updates](#transcript-updates) as it recognises speech.
1. **End** — the utterance closes (see [who ends it](#the-two-choices)), producing exactly one `UtteranceFinal` with the complete text. An utterance also closes automatically after `max_utterance_ms` (default 60 s).

`Cancel` discards an open utterance without producing a final transcript.

## Transcript updates

Every update carries a **kind** and the text so far. Treat a `Partial` as overwrite-in-place — it is always superseded by a later update.

| Kind             | Meaning                                                                                                            |
| ---------------- | ------------------------------------------------------------------------------------------------------------------ |
| `SpeechStart`    | The recogniser detected the start of speech (empty text). Offline models only.                                     |
| `Partial`        | A revisable in-progress hypothesis (`"cast"` → `"cast fire"` → `"cast fireball"`). Streaming (online) models only. |
| `SegmentFinal`   | A committed chunk while the utterance stays open.                                                                  |
| `UtteranceFinal` | The last update for the cycle; carries the complete concatenated text. Exactly one per utterance.                  |

## The two choices

How a turn behaves comes down to two **independent** choices that combine into four cases.

### 1. Who ends the utterance — push-to-talk vs hands-free

- **Push-to-talk (PTT)** — *you* decide when the utterance ends by calling `End` (e.g. on key release). Set `autoFinishOnSilence = false`. Best when the player holds a button to talk.
- **Hands-free** — the *server* closes the utterance when the player stops talking. Set `autoFinishOnSilence = true` (the default). Best for always-listening NPCs.

### 2. Which recogniser — offline vs online

The STT model's family decides how speech is segmented and whether you get partials. You don't set this directly — it comes from the model you pick.

|               | Offline (e.g. Whisper, SenseVoice, Parakeet)                         | Online / streaming (e.g. streaming Zipformer, Paraformer, T-One) |
| ------------- | -------------------------------------------------------------------- | ---------------------------------------------------------------- |
| Partials      | none                                                                 | yes (revisable, low latency)                                     |
| `SpeechStart` | yes                                                                  | no                                                               |
| Segmentation  | **Silero VAD** (voice-activity detection) slices speech from silence | the recogniser's **built-in endpointer**                         |
| Feel          | segment-lagged (decodes a chunk once it's complete)                  | streaming (updates as you speak)                                 |

**Offline** models attend over a whole window at once, so Tryll wraps them with **Silero VAD** to cut the stream into speech segments. **Online** models decode incrementally and have their own endpointer — Tryll does not layer VAD on top of them.

### How the two combine

| PTT/hands-free × recogniser | What closes the turn                                                                                                                        |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **PTT + offline**           | your `End` call (or the timeout)                                                                                                            |
| **PTT + online**            | your `End` call (partials stream meanwhile)                                                                                                 |
| **Hands-free + offline**    | the first VAD speech segment                                                                                                                |
| **Hands-free + online**     | the first non-empty endpointer result; brief pauses emit empty "heartbeat" segments that keep the turn armed so the player can keep talking |

## Sending the transcript to an agent

The server never starts a turn from speech. When a Unity or Unreal voice component has a target agent, the component sends the non-empty `UtteranceFinal` text to that agent with `SendMessage`, and drops it if the agent already has a turn in flight. Without a voice component, call `SendMessage` with the final text yourself. Either way the spoken words drive a normal turn (streamed `AnswerText`, and TTS if the graph has a [`Speak`/`GenerateAndSpeak`](https://docs.tryllengine.com/v0.8/concepts/tts-and-voice-output.html) node). Open the next utterance *after* the reply and any TTS playback finish, so the mic doesn't transcribe the agent's own voice.

## Choosing a model

STT models are declared in the catalog like any other, with `model_type: "stt"` and `engine: "sherpa-onnx"`; you reference one by name when creating the handle. Rough guidance:

- **Latency-sensitive / always-listening** → a streaming model (partials give immediate feedback).
- **Best accuracy, short commands** → an offline model (Whisper family, Parakeet).
- **Hotword biasing** (NPC/spell names) is supported by transducer/CTC families (Parakeet, streaming Zipformer/Paraformer, T-One) — **not** by Whisper or SenseVoice.

Every offline model also loads a **VAD** model (default `Silero VAD`) as a dependency; VAD entries are download/config only and are not separately pinnable. See [Model Management](https://docs.tryllengine.com/v0.8/reference/model-management.html) and, for on-disk sizes, [Estimate Memory Footprint](https://docs.tryllengine.com/v0.8/how-to/estimate-memory-footprint.html).

## Related

- [How-to: Use Voice Input](https://docs.tryllengine.com/v0.8/how-to/use-voice-input.html) — the full API walkthrough.
- [How-to: Bias Voice Input with Hotwords](https://docs.tryllengine.com/v0.8/how-to/use-voice-input-hotwords.html)
- [Concept: TTS and Voice Output](https://docs.tryllengine.com/v0.8/concepts/tts-and-voice-output.html) — the other half of a voiced NPC.
- [Concept: Models and Inference Engines](https://docs.tryllengine.com/v0.8/concepts/models-and-inference-engines.html)
- [Reference: Error Codes — 4100–4107](https://docs.tryllengine.com/v0.8/reference/error-codes.html#4xxx-inference-workflow) — voice-input errors.
