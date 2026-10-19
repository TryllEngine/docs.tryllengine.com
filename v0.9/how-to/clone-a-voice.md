# Clone a Voice from an Audio Sample

Give an NPC a custom voice from a short recording. Voice-cloning TTS models condition synthesis on a *reference clip*: a recording of the target voice. There is no training or preprocessing step. You put a WAV file in your project's storage folder and point the agent's [voice profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html) at it with **Reference Voice**.

| Model                               | Languages                                             | Clip length               | Status |
| ----------------------------------- | ----------------------------------------------------- | ------------------------- | ------ |
| `Pocket TTS (int8)`                 | English                                               | 10–20 s (first 30 s used) | Public |
| `Qwen3-TTS 12Hz 1.7B Base (Q4_K_M)` | zh, en, de, it, pt, es, ja, ko, fr, ru (via **Lang**) | 3–10 s                    | Public |

The same mechanism also powers plain **voice selection**: ship several reference clips and give each NPC a voice profile that names one. One loaded cloning model serves many distinct voices concurrently.

Prerequisites

- Voice output already working — see [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.9/how-to/add-voice-to-an-agent.html).
- A TTS model with voice cloning in the catalog (see the table above).
- A process storage root (the server's storage folder) — see [Ship Storage Folders for Builds](https://docs.tryllengine.com/v0.9/how-to/ship-storage-folders-for-builds.html).

## One voice per character

Cloning is how a character keeps **the same voice in every sentence and every turn**. The clip fixes the speaker; the model only adds the words. That makes cloning the right tool for named characters, and a better fit than models that invent a voice from a text description. Those models (for example Qwen3-TTS VoiceDesign) sample a new voice on each synthesis call, and Tryll synthesizes a reply sentence by sentence, so the voice would change mid-reply. Use them to *create* clips, not to speak at runtime.

**The clip's mood carries over.** Cloning copies the register and energy of the clip along with the timbre: an angry clip gives an angry-sounding voice. Record the base clip in the character's usual mood. To switch moods at runtime, give the profile one clip per mood — see [Add Emotions to a Voice](https://docs.tryllengine.com/v0.9/how-to/add-emotions-to-a-voice.html).

## Where clips come from

- **Record an actor.** This gives the best quality and one true identity across every mood. Follow [What to record](#what-to-record) below.
- **Design a voice with a text prompt, outside Tryll.** Qwen3-TTS VoiceDesign turns a description such as "Man in his thirties, Eastern European accent, irritated" into speech. Render a few takes, keep the one you like, and save it as the character's clip. See [Design a Voice Clip with Qwen3-TTS](https://docs.tryllengine.com/v0.9/how-to/design-a-voice-with-qwen3-tts.html).
- **Use stock or synthetic voice libraries,** as long as the licence covers use in your product.

## 1. Record (or choose) a reference clip

- **Pocket TTS: 10–20 seconds** of the target voice is the sweet spot. **Qwen3-TTS Base** reduces the clip to a single speaker fingerprint, so **3–10 seconds** is enough. A longer clip does not help it.
- For Pocket TTS, only the **first 30 seconds** of the file condition the voice — anything after that is ignored, so **put the best, continuous speech at the very start** and cut leading silence, breaths, or intros. A clip whose opening seconds carry little speech produces weak conditioning, audible as the voice drifting between sentences.
- **One speaker, clean audio.** The clip's recording quality is reproduced in the synthesized speech — background noise, reverb, and compression artifacts all carry over. Record in a quiet room.
- **Mono 16-bit PCM WAV**, ideally at 24 kHz (other sample rates are resampled automatically).

### What to record

There is **no required script**: the voice encoder only looks at the audio — no transcript is involved — so any text works. What you say matters far less than *how* you say it:

- **Record in character — style transfers.** The clip's tone, pace, energy, and emotion carry over into everything the NPC says. A weary old wizard should be read slowly and gravelly; an energetic merchant bright and fast. A flat, monotone read produces a flat, monotone NPC.
- **Speak naturally and continuously**, with a bit of pitch movement — mix a statement, a question, some emphasis. Short natural pauses are fine; long gaps, throat-clearing, and "umm, okay, so…" intros waste the 30-second window.
- **Vary the wording.** A few different sentences beat repeating one phrase. If you want a standard script, phonetically balanced passages like the *Rainbow Passage* or a handful of Harvard sentences fill 20–30 s nicely — but ordinary conversational text with varied vocabulary is nearly as good.
- **Record in the language you'll synthesize** (English for Pocket TTS). A speaker's accent carries over, which can be exactly what you want for a character. Qwen3-TTS Base speaks any of its 10 languages from one clip; set **Lang** on the voice profile.

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

## 2. Put the clip in the storage folder

**Reference Voice** is a path **relative to the process storage root**, set when the server starts — the same root used by string storages and knowledge bases:

```
<storage root>/
  └─ voices/
      └─ wizard_eldrin.wav
```

The clip lives with the game on the machine running the Tryll server. It never crosses the network — the client only sends the path.

## 3. Point the agent at the clip

### Unity

1. Set the storage root once, if you haven't already: **Edit → Project Settings → Tryll Client → Storage Data Folder**. Create a `voices/` subfolder there and drop your WAV in.
1. Create a voice profile: right-click in the Project window → **Create → Tryll → Voice Profile**.
1. Set **Tts Model Name** to `Pocket TTS (int8)` from the model dropdown. If it isn't listed, register it first in the [Model Manager](https://docs.tryllengine.com/v0.9/how-to/manage-models-in-editor.html) window.
1. In **Reference Voice**, click the **…** browse button and pick the WAV. In the file dialog, switch the extension filter to **All files** to see `.wav` files. The field stores the storage-relative path (e.g. `voices/wizard_eldrin.wav`) — you can also type it directly.
1. Assign the profile to the **Voice** field of the character's `TryllAgentComponent`.
1. Enter Play mode and send a message — the agent speaks in the cloned voice through its `TryllSpeaker`.

### Unreal

1. Set the storage root once in **Project Settings → Plugins → Tryll Client → Storage → Storage Data Folder**, create `voices/` there and drop your WAV in.
1. Create a **Tryll Voice Profile** data asset and pick `Pocket TTS (int8)` from the model dropdown (register it in the [Model Manager](https://docs.tryllengine.com/v0.9/how-to/manage-models-in-editor.html) panel if missing).
1. In **Reference Voice**, click the **...** button and select the WAV — switch the file dialog's filter to **All files (\*.\*)** to see `.wav` files. The stored value is the storage-relative path.
1. Assign the profile to the **Voice Profile** property of the character's **Tryll Agent** component.
1. Make sure the actor has a **Tryll Speaker** component, then PIE and send a message.

### Python

```
from tryll_client import VoiceProfile

wizard = VoiceProfile(
    tts_model_name="Pocket TTS (int8)",
    reference_voice="voices/wizard_eldrin.wav",  # relative to storage root
)
agent = client.create_agent(graph, voice=wizard)
```

Leave Speaker Id at 0

Cloning models are single-speaker: the voice comes entirely from the reference clip. Setting **Speaker Id** to anything other than `0` fails agent creation with an out-of-range error (`3023`).

Qwen3-TTS Base needs a clip

Pocket TTS falls back to a bundled default voice when **Reference Voice** is empty. Qwen3-TTS Base has no default voice. Without a clip, synthesis fails with an error telling you to set the voice profile's `reference_voice`.

### Switching moods at runtime

A profile can hold one clip per mood as named **delivery modes**, and game code switches between them with `SetAgentDeliveryMode("angry")`, from the next sentence. The full recipe, from making the clips to switching at runtime, is in [Add Emotions to a Voice](https://docs.tryllengine.com/v0.9/how-to/add-emotions-to-a-voice.html).

To give the character a **different voice** rather than a different mood, for example another actor, replace the whole profile with `SetAgentVoice` while the agent is idle.

Leaving **Reference Voice** empty uses the model's default voice (Pocket TTS only).

### A different clip per language

Put a per-language clip in a `loc/<locale>/` folder next to the one you linked, for example `voices/loc/de/mack.wav` beside `voices/mack.wav`. With the session locale set to `de`, the voice uses it automatically; without one, it falls back to the linked clip. See [Ship in Another Language](https://docs.tryllengine.com/v0.9/how-to/ship-in-another-language.html).

## 4. Verify

Send a message and listen: the agent speaks new text in the reference clip's voice, consistently across sentences and turns. Two agents with different **Reference Voice** clips speak in different voices at the same time on the same loaded model.

If the voice drifts between sentences, the reference clip is conditioning too weakly — almost always because its first seconds are silence or sparse speech. Re-trim so continuous speech starts at 0:00 (the server log's per-clip warning tells you when only part of a long clip is used).

## Limitations

- **Pocket TTS is English only.** The sherpa-onnx Pocket TTS conversion covers Kyutai's English model. Qwen3-TTS Base clones in 10 languages. Supertonic 3 covers 31 languages with fixed voices — see [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.9/how-to/add-voice-to-an-agent.html).
- **Mood comes from the clip, not from a parameter.** Cloning models do not take a **Voice Description** or a delivery-mode **Instruction**; setting either fails with `3023`. Use one delivery mode per mood, each with its own clip (see [Add Emotions to a Voice](https://docs.tryllengine.com/v0.9/how-to/add-emotions-to-a-voice.html)). Maya1 is the voice that takes delivery as text.
- A bad or missing clip path, in the profile or in any delivery mode, fails agent creation (or `SetAgentVoice`) with `3023`, and the message names the field.
- Cloning quality tracks clip quality. If the voice sounds muffled or noisy, re-record the reference clip.

## Related

- [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.9/how-to/add-voice-to-an-agent.html)
- [Design a Voice Clip with Qwen3-TTS](https://docs.tryllengine.com/v0.9/how-to/design-a-voice-with-qwen3-tts.html) — make a clip from a text description
- [Add Emotions to a Voice](https://docs.tryllengine.com/v0.9/how-to/add-emotions-to-a-voice.html) — one clip per mood, switched at runtime
- [Reference: Voice Profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html) — delivery modes and what each model does with them
- [Ship Storage Folders for Builds](https://docs.tryllengine.com/v0.9/how-to/ship-storage-folders-for-builds.html)
- [Concept: TTS and Voice Output](https://docs.tryllengine.com/v0.9/concepts/tts-and-voice-output.html)
- [Reference: TTS Models](https://docs.tryllengine.com/v0.9/reference/tts-models.html) — which families clone (Pocket TTS, Qwen3-TTS Base) and which take text-described voices (Maya1)
