# Use Voice Input

Voice Input is the Tryll subsystem that streams raw PCM audio to the server, runs it through a speech-to-text (STT) model, and delivers transcripts back to your application. This page covers the full lifecycle: creating a `VoiceInput` handle, starting and finishing utterances, streaming audio, and cleaning up. It ends with links to hotwords biasing and model selection.

For the mental model behind this — the utterance lifecycle, push-to-talk vs. hands-free, offline vs. streaming recognisers, and the transcript-update kinds — see [Concept: STT and Voice Input](https://docs.tryllengine.com/v0.8/concepts/stt-and-voice-input.html).

## Prerequisites

- A connected session configured with an STT engine:

```
// C++ — enable Sherpa-ONNX for STT. CreateSession takes a SessionConfig.
client.CreateSession({
    .engine    = ::Tryll::InferenceEngine_LlamaCpp,     // LLM engine
    .sttEngine = ::Tryll::InferenceEngine_SherpaOnnx,   // STT engine
});
```

```
# Python
client.create_session(
    engine=InferenceEngine.LlamaCpp,
    stt_engine=InferenceEngine.SherpaOnnx,
)
```

```
// Unity (C#) — positional; use named args to skip the ones you don't need.
client.CreateSession(
    TryllInferenceEngine.LlamaCpp,
    sttEngine: TryllInferenceEngine.SherpaOnnx);
```

```
// Unreal (C++) — positional CreateSession on the subsystem.
Subsystem->CreateSession(
    ETryllInferenceEngine::LlamaCpp,      // Engine (LLM)
    TEXT(""),                             // GameName
    ETryllInferenceEngine::SherpaOnnx);   // SttEngine
```

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
voice_input_id = client.create_voice_input(
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

The server loads the model (or reuses a cached copy) and returns a `voice_input_id`. This id is embedded in the handle objects in C++, Unity, and Unreal; it is returned directly in Python.

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
vi->SetOnTranscriptUpdate([](const Tryll::Client::TranscriptUpdate& u) {
    if (u.kind == Tryll::TranscriptUpdateKind::UtteranceFinal)
        ProcessFinalTranscript(u.text);
});
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
opts.autoSendAgentId    = myAgent->Id();  // 0 for transcribe-only
opts.autoFinishOnSilence = true;
opts.maxUtteranceMs     = 30000;
vi->BeginUtterance(opts);
```

**Python** — Python currently requires constructing the request manually via the wire codec (`encode_begin_utterance_request` in `tryll_client.codec`).

**Unity (C#)**

```
var opts = new UtteranceOptions
{
    AutoSendAgentId    = agentId,   // 0 = transcribe-only
    AutoFinishOnSilence = true,
    MaxUtteranceMs     = 30000,
};
voice.BeginUtterance(opts);
```

**Unreal (C++)**

```
FTryllUtteranceOptions Opts;
Opts.AutoSendAgentId     = AgentId;  // 0 = transcribe-only
Opts.bAutoFinishOnSilence = true;
Opts.MaxUtteranceMs      = 30000;
Voice->BeginUtterance(Opts);
```

When `autoSendAgentId` is non-zero the server automatically forwards the final transcript as a `SendMessageRequest` to that agent when the utterance closes — no extra round-trip needed.

## 4. Stream audio

Send PCM chunks as fast as they arrive. The buffer is fire-and-forget: no acknowledgement, no backpressure from the server.

**C++**

```
// Called from your audio capture callback at ~20 ms intervals.
vi->SendAudioBuffer(pcmChunk.data(), pcmChunk.size());
```

**Unity (C#)**

```
// Called from Unity's OnAudioFilterRead or a microphone polling coroutine.
voice.SendAudioBuffer(pcmBytes);
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
vi->EndUtterance();   // C++ — commit; produces UtteranceFinal
vi->CancelUtterance(); // C++ — discard; no UtteranceFinal fired
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

Keep `autoFinishOnSilence = true`. The server commits each phrase on silence and fires `UtteranceFinal` when a non-empty phrase is recognized — closing the turn and (if `autoSendAgentId` is set) auto-sending the transcript. Empty/silence finals never close the turn or autosend; during silence the server emits empty `SegmentFinal` heartbeats you can surface as a "listening…" hint. Open a fresh `BeginUtterance` for the next turn — typically after the agent's reply and any TTS playback finish, which keeps the mic from transcribing its own output.

## 7. Destroy the handle

Always destroy the `VoiceInput` when you are done with it. This frees the model slot on the server.

```
vi->Destroy();  // C++
```

```
voice.Dispose();  // Unity (C#)
```

```
Subsystem->RequestDestroyVoiceInput(Voice);  // Unreal (C++)
```

## Next steps

- [Bias Voice Input with Hotwords](https://docs.tryllengine.com/v0.8/how-to/use-voice-input-hotwords.html) — nudge the STT decoder toward game-specific proper nouns and spell names without retraining.
- [Pin and Unpin Models](https://docs.tryllengine.com/v0.8/how-to/pin-and-unpin-models.html) — keep the STT model warm between sessions to avoid reload latency.

## Related reference

- `CreateVoiceInputRequest` — wire message sent to start a VoiceInput session.
- `BeginUtteranceRequest` — wire message sent to open an audio capture window.
- `WireTranscriptUpdate` — server push carrying incremental and final transcript text.
