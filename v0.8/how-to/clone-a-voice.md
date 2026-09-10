# Clone a Voice from an Audio Sample

Give an NPC a custom voice from a short recording. Voice-cloning TTS models (currently **Pocket TTS**) condition synthesis on a *reference clip*: a recording of the target voice. There is no training or preprocessing step — you put a WAV file in your project's storage folder and point the node's **Tts Voice** parameter at it.

The same mechanism also powers plain **voice selection**: ship several reference clips and each agent (or each NPC) picks one. One loaded Pocket TTS model serves many distinct voices concurrently.

Prerequisites

- Voice output already working — see [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.8/how-to/add-voice-to-an-agent.html).
- A TTS model with voice cloning in the catalog: `Pocket TTS (int8)`.
- A session storage folder — see [Ship Storage Folders for Builds](https://docs.tryllengine.com/v0.8/how-to/ship-storage-folders-for-builds.html).

## 1. Record (or choose) a reference clip

- **10–30 seconds** of the target voice. Only the **first 30 seconds** of the file condition the voice — anything after that is ignored, so **put the best, continuous speech at the very start** and cut leading silence, breaths, or intros. A clip whose opening seconds carry little speech produces weak conditioning, audible as the voice drifting between sentences.
- **One speaker, clean audio.** The clip's recording quality is reproduced in the synthesized speech — background noise, reverb, and compression artifacts all carry over. Record in a quiet room.
- **Mono 16-bit PCM WAV**, ideally at 24 kHz (other sample rates are resampled automatically).

### What to record

There is **no required script**: the voice encoder only looks at the audio — no transcript is involved — so any text works. What you say matters far less than *how* you say it:

- **Record in character — style transfers.** The clip's tone, pace, energy, and emotion carry over into everything the NPC says. A weary old wizard should be read slowly and gravelly; an energetic merchant bright and fast. A flat, monotone read produces a flat, monotone NPC.
- **Speak naturally and continuously**, with a bit of pitch movement — mix a statement, a question, some emphasis. Short natural pauses are fine; long gaps, throat-clearing, and "umm, okay, so…" intros waste the 30-second window.
- **Vary the wording.** A few different sentences beat repeating one phrase. If you want a standard script, phonetically balanced passages like the *Rainbow Passage* or a handful of Harvard sentences fill 20–30 s nicely — but ordinary conversational text with varied vocabulary is nearly as good.
- **Record in the language you'll synthesize** (English, for the current Pocket TTS model). A speaker's accent carries over — which can be exactly what you want for a character.

Iteration is free and fast, so treat it as a loop: record ~40 s in character → trim so strong speech starts at 0:00 → generate a line → listen → re-record if the delivery isn't right.

Convert with whichever tool you already use:

1. Open the recording (any format).
1. **Tracks → Mix → Mix Stereo Down to Mono** (if stereo).
1. Cut everything before the speech starts; keep the best ≤ 30 s.
1. **File → Export Audio…** → format **WAV**, encoding **Signed 16-bit PCM**.

```
ffmpeg -i recording.mp3 -ac 1 -ar 24000 -sample_fmt s16 ^
    -af "silenceremove=start_periods=1:start_threshold=-40dB" -t 30 voice.wav
```

(`-af silenceremove…` strips leading silence; `-t 30` keeps the first 30 seconds.)

The server log prints each clip's duration when it loads and warns when only part of it is used — check it if cloning sounds off.

Consent and rights

A reference clip captures a real person's voice. Make sure you have the speaker's consent and the rights to use the recording in your product. Tryll does not verify or restrict clips — this is your content responsibility, exactly like any other licensed asset.

Model attribution

The Pocket TTS weights are [Kyutai's](https://kyutai.org/blog/2026-01-13-pocket-tts/), licensed **CC-BY-4.0**: shipping them inside your game requires crediting Kyutai (e.g. in your credits screen or license notices).

## 2. Put the clip in the session storage folder

The **Tts Voice** parameter is a path **relative to the session storage root** — the same root used by string storages and knowledge bases:

```
<storage root>/
  └─ voices/
      └─ wizard_eldrin.wav
```

The clip lives with the game on the machine running the Tryll server. It never crosses the network — the client only sends the path.

## 3. Point the agent at the clip

### Unity

1. Set the storage root once, if you haven't already: **Edit → Project Settings → Tryll Client → Storage Data Folder**. Create a `voices/` subfolder there and drop your WAV in.
1. On your `TryllWorkflowAsset`, select the **GenerateAndSpeak** (or **Speak**) node.
1. Set **Tts Model Name** to `Pocket TTS (int8)` from the model dropdown. If it isn't listed, register it first in the [Model Manager](https://docs.tryllengine.com/v0.8/how-to/manage-models-in-editor.html) window.
1. In **Tts Voice**, click the **…** browse button and pick the WAV. In the file dialog, switch the extension filter to **All files** to see `.wav` files. The field stores the storage-relative path (e.g. `voices/wizard_eldrin.wav`) — you can also type it directly.
1. Enter Play mode and send a message — the agent speaks in the cloned voice through its `TryllSpeaker`.

From code instead of the Inspector:

```
.AddGenerateAndSpeak("speak", new TryllGenerateAndSpeakParams
{
    TtsModelName = "Pocket TTS (int8)",
    TtsVoice     = "voices/wizard_eldrin.wav",  // relative to storage root
    DefaultExit  = "",
})
```

### Unreal

1. Set the storage root once in **Project Settings → Tryll → StorageDataFolder**, create `voices/` there and drop your WAV in.
1. Open your **Tryll Workflow Asset** and select the **GenerateAndSpeak** (or **Speak**) node.
1. Enable **Override Tts Model Name** and pick `Pocket TTS (int8)` from the dropdown (register it in the [Model Manager](https://docs.tryllengine.com/v0.8/how-to/manage-models-in-editor.html) panel if missing).
1. In **Tts Voice**, click **Browse** and select the WAV — switch the file dialog's filter to **All files (\*.\*)** to see `.wav` files. The stored value is the storage-relative path.
1. Make sure the actor has a **Tryll Speaker** component, then PIE and send a message.

From C++:

```
auto* SpeakParams = NewObject<UTryllGenerateAndSpeakParams>(this);
SpeakParams->bOverrideTtsModelName = true;
SpeakParams->TtsModelName          = TEXT("Pocket TTS (int8)");
SpeakParams->TtsVoice.Path         = TEXT("voices/wizard_eldrin.wav");
```

Leave Speaker Id at 0

Pocket TTS is a single-speaker model — the voice comes entirely from the reference clip. Setting **Speaker Id** to anything other than `0` fails agent creation with an out-of-range error.

### Switching voices at runtime

**Tts Voice** is **mutable** — change it between turns with [Change Agent Parameters](https://docs.tryllengine.com/v0.8/how-to/change-agent-parameters.html), no agent rebuild needed. The first turn with a new clip computes a voice embedding (fast); afterwards it is cached.

Leaving **Tts Voice** empty uses the model's default voice.

## 4. Verify

Send a message and listen: the agent speaks new text in the reference clip's voice, consistently across sentences and turns. Two agents with different **Tts Voice** clips speak in different voices at the same time on the same loaded model.

If the voice drifts between sentences, the reference clip is conditioning too weakly — almost always because its first seconds are silence or sparse speech. Re-trim so continuous speech starts at 0:00 (the server log's per-clip warning tells you when only part of a long clip is used).

## Limitations

- **English only for now.** The sherpa-onnx Pocket TTS conversion currently covers Kyutai's English model. For other languages, use Supertonic 3 with the **Tts Lang** parameter (fixed voice, 31 languages) — see [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.8/how-to/add-voice-to-an-agent.html).
- A bad or missing clip path fails agent creation with a clear error; if the path is broken later by a runtime parameter change, the turn logs an error and falls back to the default voice.
- Cloning quality tracks clip quality. If the voice sounds muffled or noisy, re-record the reference clip.

## Related

- [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.8/how-to/add-voice-to-an-agent.html)
- [Change Agent Parameters at Runtime](https://docs.tryllengine.com/v0.8/how-to/change-agent-parameters.html)
- [Ship Storage Folders for Builds](https://docs.tryllengine.com/v0.8/how-to/ship-storage-folders-for-builds.html)
- [Concept: TTS and Voice Output](https://docs.tryllengine.com/v0.8/concepts/tts-and-voice-output.html)
- [Reference: TTS Models](https://docs.tryllengine.com/v0.8/reference/tts-models.html) — Pocket TTS is the only cloning-capable family Tryll supports
