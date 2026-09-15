# TryllVoiceInputComponent

**Type:** `MonoBehaviour`\
**Namespace:** `Tryll.Client`\
**Source:** `Runtime/TryllVoiceInputComponent.cs`

Manages one `TryllVoiceInput` handle and drives it from the Unity `Microphone` API. It captures PCM audio in a coroutine and streams it to the Tryll server for speech-to-text transcription. Transcripts can be forwarded automatically to a [`TryllAgentComponent`](https://docs.tryllengine.com/reference/client-api/unity/tryll-agent-component.html).

______________________________________________________________________

## Inspector fields

| Field                 | Type                  | Default                    | Description                                                                         |
| --------------------- | --------------------- | -------------------------- | ----------------------------------------------------------------------------------- |
| `modelName`           | `string`              | `"Whisper Tiny EN (int8)"` | STT model catalog name.                                                             |
| `targetAgent`         | `TryllAgentComponent` | —                          | Agent that receives the final transcript. Leave empty for transcribe-only mode.     |
| `autoFinishOnSilence` | `bool`                | `true`                     | Let the server VAD close the utterance on silence (stop-on-silence mode).           |
| `maxUtteranceMs`      | `uint`                | `60000`                    | Hard timeout per utterance in milliseconds.                                         |
| `vadThreshold`        | `float`               | `0.5`                      | Silero VAD speech-probability threshold (0–1).                                      |
| `vadMinSilenceMs`     | `uint`                | `500`                      | Silence duration in ms that closes a speech segment.                                |
| `vadSpeechPadMs`      | `uint`                | `250`                      | Padding in ms added around detected speech.                                         |
| `micDeviceName`       | `string`              | `""`                       | Microphone device name. Empty = system default.                                     |
| `pumpIntervalMs`      | `float`               | `50`                       | PCM pump cadence in milliseconds.                                                   |
| `ringBufferSeconds`   | `int`                 | `2`                        | Microphone ring-buffer length in seconds.                                           |
| `createOnEnable`      | `bool`                | `true`                     | Create the VoiceInput handle automatically once the `TryllClient` session is ready. |

______________________________________________________________________

## Runtime state

```
public TryllVoiceInput Voice       { get; }
public bool HasVoiceInput          { get; }
public bool IsUtteranceActive      { get; }
```

______________________________________________________________________

## Public API

```
// Handle lifecycle
public async void CreateVoiceInput();
public void       DestroyVoiceInput();

// Utterance control
public void BeginUtterance(TryllAgentComponent agentOverride = null);
public void EndUtterance();
public void CancelUtterance();
```

`BeginUtterance` starts the microphone and opens a server-side utterance. When `agentOverride` is given the final transcript is sent to that agent; otherwise `targetAgent` is used. When both are null the component operates in transcribe-only mode.

`EndUtterance` commits the current utterance (push-to-talk release) and stops capture.

`CancelUtterance` discards the current utterance without producing a transcript.

______________________________________________________________________

## Events

| Event                 | Signature                      | Description                                                                                     |
| --------------------- | ------------------------------ | ----------------------------------------------------------------------------------------------- |
| `OnTranscriptUpdate`  | `UnityEvent<TranscriptUpdate>` | Fired for each transcript update (partial and final). Check `update.Kind` for `UtteranceFinal`. |
| `OnError`             | `UnityEvent<TryllError>`       | Fired on protocol-level errors.                                                                 |
| `OnVoiceInputCreated` | `UnityEvent`                   | Fired when the VoiceInput handle is created.                                                    |

______________________________________________________________________

## Capture model

- Microphone starts on `BeginUtterance` and stops when the `UtteranceFinal` arrives (or on `EndUtterance` / `CancelUtterance`). Each turn closes on a non-empty speech final; begin a new utterance for the next turn.
- A coroutine pumps small PCM chunks at `pumpIntervalMs` cadence into the server's audio pipeline.
- The device-native sample rate is declared to the server; the server resamples to the STT model's expected rate.

______________________________________________________________________

## Lifetime and ownership

- The component owns its `TryllVoiceInput` handle and disposes it in `OnDisable` and `OnDestroy`.
- On disconnect the component drops its local state without sending `DestroyVoiceInput` (the TCP connection is gone).

______________________________________________________________________

## See also

- [`TryllAgentComponent`](https://docs.tryllengine.com/reference/client-api/unity/tryll-agent-component.html) — receives the final transcript
- [`TryllClient`](https://docs.tryllengine.com/reference/client-api/unity/tryll-client.html) — session manager
- [Tool calling](https://docs.tryllengine.com/concepts/tool-calling.html)
