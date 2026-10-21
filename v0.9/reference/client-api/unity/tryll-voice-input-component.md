# TryllVoiceInputComponent

**Type:** `MonoBehaviour`\
**Namespace:** `Tryll.Client`\
**Source:** `Runtime/TryllVoiceInputComponent.cs`

Manages one `TryllVoiceInput` handle and drives it from the Unity `Microphone` API. It captures PCM audio in a coroutine and streams it to the Tryll server for speech-to-text transcription. Transcripts can be forwarded automatically to a [`TryllAgentComponent`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-agent-component.html).

______________________________________________________________________

## Inspector fields

| Field                 | Type                  | Default                    | Description                                                                                                                                                                                          |
| --------------------- | --------------------- | -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `modelName`           | `string`              | `"Whisper Tiny EN (int8)"` | STT model catalog name.                                                                                                                                                                              |
| `targetAgent`         | `TryllAgentComponent` | —                          | Agent that receives the final transcript. Leave empty for transcribe-only mode.                                                                                                                      |
| `autoFinishOnSilence` | `bool`                | `true`                     | Let the server VAD close the utterance on silence (stop-on-silence mode).                                                                                                                            |
| `maxUtteranceMs`      | `uint`                | `60000`                    | Hard timeout per utterance in milliseconds.                                                                                                                                                          |
| `vadThreshold`        | `float`               | `0.5`                      | Silero VAD speech-probability threshold (0–1).                                                                                                                                                       |
| `vadMinSilenceMs`     | `uint`                | `1000`                     | Silence duration in ms that closes a speech segment. A shorter value splits one spoken sentence at ordinary pauses, and each fragment is then recognized without the context of the rest.            |
| `vadSpeechPadMs`      | `uint`                | `250`                      | Padding in ms added around detected speech.                                                                                                                                                          |
| `hotwordsStoragePath` | `string`              | `""`                       | Relative path (resolved against Storage Data Folder) to a UTF-8 phrase list. Empty disables hotword biasing. Inspector browse uses the same file picker as knowledge-base and canned-response paths. |
| `hotwordsScore`       | `float`               | `1.5`                      | How strongly matched phrases are weighted against one another during beam search. Ignored when `hotwordsStoragePath` is empty.                                                                       |
| `micDeviceName`       | `string`              | `""`                       | Microphone device name. Empty = system default.                                                                                                                                                      |
| `pumpIntervalMs`      | `float`               | `50`                       | PCM pump cadence in milliseconds.                                                                                                                                                                    |
| `ringBufferSeconds`   | `int`                 | `2`                        | Microphone ring-buffer length in seconds.                                                                                                                                                            |
| `createOnEnable`      | `bool`                | `true`                     | Create the VoiceInput handle automatically once the `TryllClient` session is ready.                                                                                                                  |

______________________________________________________________________

## Runtime state

```
public TryllVoiceInput Voice       { get; }
public bool HasVoiceInput          { get; }
public bool IsUtteranceActive      { get; }

// Microphone check (see "No microphone" below)
public TryllMicrophoneStatus MicrophoneStatus { get; }  // Unknown, Available, NoDevice, DeviceNotFound, CaptureFailed
public string MicrophoneStatusMessage         { get; }  // why it is unavailable; empty otherwise
public bool IsMicrophoneUnavailable           { get; }  // NoDevice, DeviceNotFound or CaptureFailed
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

`CreateVoiceInput` and `BeginUtterance` both check the microphone first. Without one they raise `OnError` with `MicrophoneUnavailable` (`4108`) and send nothing to the server.

______________________________________________________________________

## Events

| Event                 | Signature                      | Description                                                                                                   |
| --------------------- | ------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| `OnTranscriptUpdate`  | `UnityEvent<TranscriptUpdate>` | Fired for each transcript update (partial and final). Check `update.Kind` for `UtteranceFinal`.               |
| `OnError`             | `UnityEvent<TryllError>`       | Fired on protocol-level errors, and with `MicrophoneUnavailable` (`4108`) when there is no usable microphone. |
| `OnVoiceInputCreated` | `UnityEvent`                   | Fired when the VoiceInput handle is created.                                                                  |

______________________________________________________________________

## Capture model

- Microphone starts on `BeginUtterance` and stops when the `UtteranceFinal` arrives (or on `EndUtterance` / `CancelUtterance`). Each turn closes on a non-empty speech final; begin a new utterance for the next turn.
- A coroutine pumps small PCM chunks at `pumpIntervalMs` cadence into the server's audio pipeline.
- The device-native sample rate is declared to the server; the server resamples to the STT model's expected rate.

### No microphone

Before creating the voice input and before every utterance, the component checks `Microphone.devices`:

| `MicrophoneStatus` | Cause                                                                                                                                     |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `NoDevice`         | The system reports no recording device.                                                                                                   |
| `DeviceNotFound`   | `micDeviceName` names a device that is not connected. The message lists the devices that are.                                             |
| `CaptureFailed`    | The device is listed but `Microphone.Start` could not open it (in use by another app, or blocked by Windows microphone privacy settings). |

In each case the voice input is not created, so no STT model is loaded; one `[Tryll]` warning is logged per distinct problem; and every `CreateVoiceInput` / `BeginUtterance` raises `OnError` with `MicrophoneUnavailable` (`4108`) and the same message. Text chat on the agent is unaffected.

The automatic create (`createOnEnable`) is the exception: it can run inside `OnEnable`, before other scripts subscribe to `OnError`, so it only sets the status and logs. Read `MicrophoneStatus` at start-up rather than waiting for an error.

Read `IsMicrophoneUnavailable` to show "type to chat" instead of a listening prompt. Calling `BeginUtterance` again is how to recover: it re-reads the device list and, once a microphone is present, creates the voice input and reports `VoiceInputStarting` (`4109`) for that one call: not a failure, just begin again a moment later. Re-check on a user action or a timer, not every frame.

______________________________________________________________________

## Lifetime and ownership

- The component owns its `TryllVoiceInput` handle and disposes it in `OnDisable` and `OnDestroy`.
- On disconnect the component drops its local state without sending `DestroyVoiceInput` (the TCP connection is gone).

______________________________________________________________________

## See also

- [`TryllAgentComponent`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-agent-component.html) — receives the final transcript
- [`TryllClient`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-client.html) — session manager
- [Bias Voice Input with Hotwords](https://docs.tryllengine.com/v0.9/how-to/use-voice-input-hotwords.html)
- [Tool calling](https://docs.tryllengine.com/v0.9/concepts/tool-calling.html)
