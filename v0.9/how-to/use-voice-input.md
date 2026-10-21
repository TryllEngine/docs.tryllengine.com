# Use Voice Input

Voice Input is the Tryll subsystem that streams raw PCM audio to the server, runs it through a speech-to-text (STT) model, and delivers transcripts back to your application. This page covers the full lifecycle: creating a `VoiceInput` handle, starting and finishing utterances, streaming audio, and cleaning up. It ends with links to hotwords biasing and model selection.

For the mental model behind this — the utterance lifecycle, push-to-talk vs. hands-free, offline vs. streaming recognisers, and the transcript-update kinds — see [Concept: STT and Voice Input](https://docs.tryllengine.com/v0.9/concepts/stt-and-voice-input.html).

## Prerequisites

- A connected session. No STT engine setting is needed: the STT model's catalog entry names its engine (Sherpa-ONNX).
- An STT model on disk — download one in the editor Model Manager, or from code (`DownloadModel` in C++, `download_model` in Python, `RequestDownloadModel` in Unity and Unreal).
- With the Unity or Unreal voice input component: a microphone. The component checks for one itself and reports when there is none; see [Handle a missing microphone](#handle-a-missing-microphone).

Optionally, warm the STT engine up at session creation so the first utterance does not pay its start-up cost:

```
// C++ — `engines` is an optional warm-up hint, not a requirement.
client.CreateSession({
    .engines = {{::Tryll::ModelRole_Stt, ::Tryll::InferenceEngine_SherpaOnnx}},
});
```

```
# Python
client.create_session(engines=[(ModelRole.Stt, InferenceEngine.SherpaOnnx)])
```

The Unity and Unreal plugins build this hint from the Model Manager's manifest when they create the session, so there is nothing to add there.

## 1. Create a VoiceInput handle

A `VoiceInput` is a stateful server-side object that holds the loaded STT model and the current utterance state. Create one per microphone source; most games need only one.

**C++**

```
#include <tryll/VoiceInput.h>

Tryll::Client::VoiceInputConfig cfg;
cfg.modelName        = "Parakeet TDT 0.6B v3 (int8)";
cfg.inputFormat      = { .sampleRate = 48000, .channels = 1, .bitsPerSample = 16 };
cfg.vadThreshold     = 0.5f;   // Silero VAD speech-probability threshold
cfg.vadMinSilenceMs  = 1000;   // silence that ends a segment (ms)
cfg.vadSpeechPadMs   = 250;    // padding around detected speech (ms)

auto vi = client.CreateVoiceInput(cfg);
```

**Python**

```
voice = client.create_voice_input(
    model_name="Parakeet TDT 0.6B v3 (int8)",
    sample_rate=48000, channels=1, bits_per_sample=16,
)
```

**Unity (C#)**

```
var cfg = new VoiceInputConfig
{
    ModelName       = "Parakeet TDT 0.6B v3 (int8)",
    InputFormat     = new AudioFormat { SampleRate = 48000, Channels = 1, BitsPerSample = 16 },
    VadThreshold    = 0.5f,
    VadMinSilenceMs = 1000,
    VadSpeechPadMs  = 250,
};
var (voice, err) = await client.CreateVoiceInputAsync(cfg);
if (!err.IsOk) { /* handle */ }
```

**Unreal (C++)**

```
FTryllVoiceInputConfig Config;
Config.ModelName              = TEXT("Parakeet TDT 0.6B v3 (int8)");
Config.InputFormat.SampleRate = 48000;

Subsystem->CreateVoiceInput(Config,
    [](TSharedPtr<FTryllVoiceInput> Voice, FTryllError Err)
    {
        if (!Err.IsOk()) { /* handle */ return; }
        // Store Voice — it is now ready for BeginUtterance.
    });
```

The server loads the model (or reuses a cached copy) and returns a `voice_input_id`. Every client wraps that id in a `VoiceInput` handle (`VoiceInput` in C++ and Python, `TryllVoiceInput` in Unity, `FTryllVoiceInput` in Unreal); you call the utterance methods on the handle.

### A note on `vadSpeechPadMs`

The voice-activity detector hands the recognizer only the part of the audio it judged to be speech, cropped tight. Recognizers are trained on recordings, which always begin with a little room tone, so a crop with no silence around it costs you the opening word — and for a very short utterance, everything. A one-word "Fireball" cropped to 0.9 seconds transcribes as nothing at all; with 100 ms of silence added to each end it transcribes correctly.

`vadSpeechPadMs` is that silence. The server adds it around each segment before decoding, so leave it at the default unless you have a reason not to. Raising it above 500 ms has no effect.

## 2. Set up transcript callbacks

Register a callback before starting utterances so you receive transcripts as they arrive. Tryll delivers four update kinds:

| Kind             | When                                               |
| ---------------- | -------------------------------------------------- |
| `SpeechStart`    | VAD detects rising edge; no text yet               |
| `Partial`        | In-progress hypothesis from online (streaming) STT |
| `SegmentFinal`   | Engine committed a chunk; utterance still open     |
| `UtteranceFinal` | Last update for this Begin/End cycle               |

**C++**

```
vi.SetOnTranscriptUpdate([](const Tryll::Client::TranscriptUpdate& u) {
    if (u.kind == ::Tryll::TranscriptUpdateKind_UtteranceFinal)
        ProcessFinalTranscript(u.text);
});
```

**Python**

```
from tryll_client.voice_input import TranscriptUpdateKind

def on_transcript(kind: int, text: str, audio_ms_consumed: int) -> None:
    if kind == TranscriptUpdateKind.UtteranceFinal:
        process_final_transcript(text)

voice.set_on_transcript_update(on_transcript)
```

**Unity (C#)**

```
voice.OnTranscriptUpdate += update =>
{
    if (update.Kind == TranscriptUpdateKind.UtteranceFinal)
        ProcessFinalTranscript(update.Text);
};
```

**Unreal (C++)**

```
Voice->OnTranscriptUpdate.AddLambda([](const FTryllTranscriptUpdate& U)
{
    if (U.Kind == ETryllTranscriptUpdateKind::UtteranceFinal)
        ProcessFinalTranscript(U.Text);
});
```

## 3. Begin an utterance

`BeginUtterance` opens a recording window and arms the VAD.

**C++**

```
Tryll::Client::UtteranceOptions opts;
opts.autoFinishOnSilence = true;
opts.maxUtteranceMs      = 30000;
vi.BeginUtterance(opts);
```

**Python** — `VoiceInput.begin_utterance(auto_finish_on_silence=..., max_utterance_ms=...)`. The transcript callback runs on the reader thread: start a turn from it with `send_message_async`. The blocking `send_message` deadlocks on that thread.

**Unity (C#)**

```
var opts = new UtteranceOptions
{
    AutoFinishOnSilence = true,
    MaxUtteranceMs      = 30000,
};
voice.BeginUtterance(opts);
```

**Unreal (C++)**

```
FTryllUtteranceOptions Opts;
Opts.bAutoFinishOnSilence = true;
Opts.MaxUtteranceMs       = 30000;
Voice->BeginUtterance(Opts);
```

The server only transcribes. On a non-empty `UtteranceFinal`, start the turn yourself: `SendText` (C++), `send_message_async` (Python, from the transcript callback), or `SendMessage` on the agent component (Unity / Unreal). The engine voice components do that when a target agent is set, and drop the final with a warning if that agent already has a turn in flight. Leave the target unset when you want to filter or rewrite the text before sending.

## 4. Stream audio

Send PCM chunks as fast as they arrive. The buffer is fire-and-forget: no acknowledgement, no backpressure from the server.

**C++**

```
// Called from your audio capture code at ~20 ms intervals.
// pcmChunk is a std::vector<std::int16_t> of mono samples.
vi.SendAudioBuffer(pcmChunk);   // binds to std::span<const std::int16_t>
```

**Python**

```
voice.send_audio_buffer(pcm_bytes)   # int16 little-endian PCM bytes
```

**Unity (C#)**

```
// Called on the main thread, e.g. from a coroutine that polls
// Microphone.GetPosition and reads new samples with AudioClip.GetData.
// SendAudioBuffer takes mono 16-bit samples (ReadOnlySpan<short>).
var pcm = new short[floatSamples.Length];
for (int i = 0; i < pcm.Length; i++)
    pcm[i] = (short)(Mathf.Clamp(floatSamples[i], -1f, 1f) * short.MaxValue);
voice.SendAudioBuffer(pcm);
```

**Unreal (C++)**

```
// Called from the audio capture thread.
Voice->SendAudioBuffer(PcmChunk);
```

Audio must be raw signed-16-bit PCM at the sample rate declared in `inputFormat`. The server resamples to the model's expected rate as needed. No encoding (opus, mp3, etc.) is supported on the voice path.

## 5. End or cancel an utterance

If `autoFinishOnSilence` is true the server closes the segment automatically. You can also close it explicitly:

```
vi.EndUtterance();    // C++ — commit; produces UtteranceFinal
vi.CancelUtterance(); // C++ — discard; no UtteranceFinal fired
```

```
voice.end_utterance()     # Python
voice.cancel_utterance()  # Python
```

```
voice.EndUtterance();    // Unity (C#)
voice.CancelUtterance(); // Unity (C#)
```

```
Voice->EndUtterance();    // Unreal (C++)
Voice->CancelUtterance(); // Unreal (C++)
```

## 6. Input modes

### Push-to-talk

Call `BeginUtterance` when the player presses a button and `EndUtterance` when they release it. Disable `autoFinishOnSilence` so VAD does not cut short a deliberate pause mid-sentence.

### Hands-free (VAD-driven)

Keep `autoFinishOnSilence = true`. The server commits each phrase on silence and fires `UtteranceFinal`. A non-empty final is the text you send (see §3). The engine voice components drop that final when the target agent already has a turn in flight. Empty finals close the utterance and do not start a turn; during silence the server emits empty `SegmentFinal` heartbeats you can surface as a "listening…" hint. Open a fresh `BeginUtterance` for the next turn — typically after the agent's reply and any TTS playback finish, which keeps the mic from transcribing its own output.

## 7. Destroy the handle

Always destroy the `VoiceInput` when you are done with it. This frees the model slot on the server.

In C++ the handle is move-only and has no `Destroy()` method: its destructor cancels any active utterance and destroys the server-side object. Let it go out of scope (or reset the object that owns it); move it to keep it alive longer.

```
voice.destroy()  # Python — or use `with client.create_voice_input(...) as voice:`
```

```
voice.Dispose();  // Unity (C#)
```

```
Subsystem->RequestDestroyVoiceInput(Voice);  // Unreal (C++)
```

## Handle a missing microphone

The Unity `TryllVoiceInputComponent` and Unreal `UTryllVoiceInputComponent` check for a microphone before creating the voice input and before every `BeginUtterance`. When there is none they do not create the voice input (no STT model is loaded), log one `[Tryll]` warning naming the component, and report `MicrophoneUnavailable` (`4108`) from each call. Text chat is unaffected. Automatic creation (on enable / `BeginPlay`) only sets the status and logs, because it can run before your listeners are bound: read the status at start-up.

- **Unity:** `OnError` receives the `TryllError`; `MicrophoneStatus` / `IsMicrophoneUnavailable` say why (`NoDevice`, `DeviceNotFound` for a `micDeviceName` that is not connected, or `CaptureFailed`).
- **Unreal:** bind `OnErrorDetailed` to get the `FTryllError` with its code (`OnError` still fires with the message alone); `GetMicrophoneStatus` / `IsMicrophoneUnavailable` say why (`NoDevice` or `CaptureFailed`).

Show "type to chat" while the microphone is unavailable. To recover, call `BeginUtterance` again on a user action or a timer: once a microphone is present it creates the voice input and reports `VoiceInputStarting` (`4109`) for that one call. In Unreal, if the default device's sample rate changes under a live voice input, `BeginUtterance` recreates the voice input with the new rate and also reports `4109`; the microphone status stays `Available`. Treat `4109` as "try again in a moment", not as an error to show.

## Next steps

- [Bias Voice Input with Hotwords](https://docs.tryllengine.com/v0.9/how-to/use-voice-input-hotwords.html) — nudge the STT decoder toward game-specific proper nouns and spell names without retraining.
- [Pin and Unpin Models](https://docs.tryllengine.com/v0.9/how-to/pin-and-unpin-models.html) — keep the STT model warm between sessions to avoid reload latency.

## Related reference

- `CreateVoiceInputRequest` — wire message sent to start a VoiceInput session.
- `BeginUtteranceRequest` — wire message sent to open an audio capture window.
- `WireTranscriptUpdate` — server push carrying incremental and final transcript text.
