# Add Voice Output to an Agent

Make an NPC speak. This guide covers every step: downloading a voice, giving the agent a [voice profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html), building a graph that produces speech — with the fused [GenerateAndSpeak](https://docs.tryllengine.com/v0.9/reference/nodes/generate-and-speak.html) node or the TTS-only [Speak](https://docs.tryllengine.com/v0.9/reference/nodes/speak.html) node — wiring up the playback component, and reacting to audio lifecycle events.

The voice belongs to the agent, not to the graph. The nodes only say *when* to speak; the voice profile says *how*. One workflow can therefore drive many characters, each with its own voice.

Use **GenerateAndSpeak** to speak a generated answer with the lowest latency (synthesis overlaps generation). Use **Speak** to voice text that did *not* come from a generation pass — e.g. a scripted [CannedResponse](https://docs.tryllengine.com/v0.9/reference/nodes/canned-response.html) — or any upstream text you want spoken without a second LLM cost. Both are wired to the client exactly the same way; only the graph differs.

For background on how streaming TTS works, see [Concept: TTS and Voice Output](https://docs.tryllengine.com/v0.9/concepts/tts-and-voice-output.html).

Prerequisites

- A connected session — see [Connect and Manage a Session](https://docs.tryllengine.com/v0.9/how-to/connect-and-manage-session.html).
- A TTS model available on the server. Acquire it beforehand — via the editor Model Manager or [Use Your Own Local Model](https://docs.tryllengine.com/v0.9/how-to/use-your-own-local-model.html) — since `CreateAgent` fails fast on a missing model rather than downloading it for you.

## 1. Download a voice

Open the Model Manager — **Window → Tryll → Model Manager** in Unity, **Window → Tryll → Tryll Model Manager** in Unreal — then download and register a TTS model, for example `Supertonic 3 (int8)` (the voice profile's model list shows registered models only). That is the whole session setup: the model's catalog entry names the engine that runs it, so there is no TTS engine to choose. The session the plugin creates already works (Unreal **Auto Create Session**, on by default; Unity `StartSession()` or **Auto Start Session**).

Upgrading from a pre-protocol-v13 version

Older versions needed **Tts Engine = SherpaOnnx** on `CreateSession` or in Project Settings, and a session left on `Mock` spoke no audio. Protocol v13 removed those fields. `CreateSession` now carries at most an optional list of `(role, engine)` hints that only warm engines early; it never chooses the engine a model runs on.

## 2. Create a voice profile

A voice profile names the TTS model and the voice. `TtsModelName` is required: there is no default voice, and a graph that speaks without a voice fails agent creation with `3025 AgentVoiceRequired`. `SpeakerId`, `Speed`, `Lang` (synthesis language on multilingual models) and `ReferenceVoice` (reference clip for voice-cloning models) are optional. Every field is listed in [Voice Profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html).

1. In the Project window: right-click → **Create → Tryll → Voice Profile**.

1. Set **Tts Model Name** from the dropdown (e.g. `Supertonic 3 (int8)`). Only registered TTS models are listed — register it first in the [Model Manager](https://docs.tryllengine.com/v0.9/how-to/manage-models-in-editor.html) if it's missing.

1. Set **Speaker Id** and **Speed** as needed.

1. Assign the profile to the **Voice** field of your `TryllAgentComponent`. For a character spawned at runtime, set `agentComponent.Voice` from code before the agent is created.

1. In the Content Browser, create a **Tryll Voice Profile** data asset (`UTryllVoiceProfile`).

1. Pick the TTS model from the dropdown (e.g. `Supertonic 3 (int8)`). Only registered TTS models are listed — register it first in the [Model Manager](https://docs.tryllengine.com/v0.9/how-to/manage-models-in-editor.html) if it's missing.

1. Set **Speaker Id** and **Speed** as needed.

1. Assign the profile to the **Voice Profile** property of your `UTryllAgentComponent`.

```
from tryll_client import VoiceProfile

voice = VoiceProfile(
    tts_model_name="Supertonic 3 (int8)",  # must match a name in models.json
    speaker_id=0,                          # voice index (0 = first style)
    speed=1.0,
)
agent = client.create_agent(graph, voice=voice)
```

```
Tryll::VoiceProfileT voice;
voice.tts_model_name = "Supertonic 3 (int8)";
voice.speaker_id     = 0;
voice.speed          = 1.0f;

Tryll::Client::AgentCreateOptions opts;
opts.voice = std::move(voice);
auto agent = client.CreateAgent(graph, std::move(opts));
```

Give each character its own profile. Two agents built from the same workflow speak in different voices when their profiles differ.

## 3. Build a graph with a GenerateAndSpeak node

Replace any `Generate` node in your graph with `GenerateAndSpeak`. The node has no voice fields; it speaks with the agent's profile. Its only TTS settings are the chunking knobs `MinSentenceChars` and `FirstChunkMaxChars`, both optional.

1. In the Project window: right-click → **Create → Tryll → Workflow Asset**.
1. Under **Graph → Nodes**, add a node named `speak` and pick **GenerateAndSpeak** in the **Params** type picker.
1. Leave **Default Exit** empty (END) unless you chain further nodes.
1. Set **Start Node** to `speak` and **Default Model Name** to your language model, then assign the asset to your `TryllAgentComponent`'s **Workflow Asset** field, next to its **Voice**.

The same graph built in code, for agents you create directly rather than through a `TryllAgentComponent`:

```
var graph = new TryllGraphBuilder()
    .AddGenerateAndSpeak("speak", new TryllGenerateAndSpeakParams
    {
        DefaultExit = "",               // END
    })
    .SetStartNode("speak")
    .SetDefaultModelName("My Language Model")
    .Build();

// voiceProfile is a TryllVoiceProfile asset (see step 2).
var (agent, error) = await TryllClient.Instance.RequestCreateAgentAsync(
    graph, voice: voiceProfile.Data);
agent.SetOnAnswerText((nodeName, text, isDelta, isFinal) => { /* stream text */ });
agent.SetOnTurnComplete((status, debugInfo, tokens) => { /* turn ended */ });

var speaker = GetComponent<TryllSpeaker>(); // or AddComponent
agent.SetOnTtsAudioFormat(speaker.HandleFormat);
agent.SetOnTtsAudio(speaker.HandleAudio);
```

1. Open or create a **Tryll Workflow Asset** in the Content Browser.
1. Add a **GenerateAndSpeak** node.
1. Leave **Default Exit** empty (END) unless you chain further nodes.
1. Set **Start Node** to `speak` and assign the asset to your `UTryllAgentComponent`, next to its **Voice Profile**.

```
auto* SpeakParams = NewObject<UTryllGenerateAndSpeakParams>(this);

FTryllGraphDescription Graph;
FTryllNodeDescription Node;
Node.Name   = TEXT("speak");
Node.Params = SpeakParams;
Graph.Nodes.Add(Node);
Graph.StartNode           = TEXT("speak");
Graph.DefaultModelName    = TEXT("My Language Model");

// Author the same graph as a UTryllWorkflowAsset and assign it, with a
// voice profile, to a UTryllAgentComponent. For agents created directly
// from a graph, see the Unreal client reference for passing the voice.
```

### Alternative: voice scripted text with a Speak node

When the text to speak does **not** come from the language model — e.g. a fixed [CannedResponse](https://docs.tryllengine.com/v0.9/reference/nodes/canned-response.html) — chain a [Speak](https://docs.tryllengine.com/v0.9/reference/nodes/speak.html) node after the node that produced it. `Speak` voices its `input`-resolved slot and runs no LLM inference, so a scripted line costs nothing to generate — set `input` to the upstream node's output slot name (empty `input` defaults to `user_message`, like every other input-bearing node, so an explicit `input` is required here). It speaks with the same voice profile as every other speaking node of the agent. `MinSentenceChars` and `FirstChunkMaxChars` behave exactly as on `GenerateAndSpeak`.

```
var graph = new TryllGraphBuilder()
    .AddCannedResponse("line", new TryllCannedResponseParams
    {
        // … canned-response config …
        DefaultExit = "say",          // route into the Speak node
    })
    .AddSpeak("say", new TryllSpeakParams
    {
        Input       = "line",         // voice the CannedResponse "line" slot
        DefaultExit = "",             // END
    })
    .SetStartNode("line")
    .Build();
```

```
auto* SpeakParams = NewObject<UTryllSpeakParams>(this);
SpeakParams->bOverrideInput = true;
SpeakParams->Input          = FTryllSlotInput(TEXT("line")); // voice the CannedResponse "line" slot
// … add a CannedResponse node named "line" routing its DefaultExit to "say",
//    then add SpeakParams as node "say" …
```

1. Add your text-producing node (e.g. **CannedResponse**) and a **Speak** node.
1. Set the producer node's **Default Exit** to the Speak node's name.
1. On the Speak node, enable **Override Input** and set **Input** to the producer node's output slot (its node name, e.g. `line`). Without this, an empty **Input** voices `user_message` instead of the scripted line.

Latency

`Speak` only sees text after the upstream node has fully finished, so chained after a `Generate` node audio begins only once generation completes. To speak a *generated* answer with minimal delay, prefer `GenerateAndSpeak`, which overlaps synthesis with generation. See [Speak](https://docs.tryllengine.com/v0.9/reference/nodes/speak.html) and [Generate and Speak](https://docs.tryllengine.com/v0.9/reference/nodes/generate-and-speak.html).

## 4. Add the speaker component

### Unity

`TryllAgentComponent` auto-provisions a `TryllSpeaker` at `CreateAgent` time if the graph contains a `GenerateAndSpeak` **or** `Speak` node and no speaker was assigned. In the common case — one NPC, one agent — you don't need to do anything extra:

```
GameObject
  ├─ TryllAgentComponent   ← graph with GenerateAndSpeak
  └─ TryllSpeaker          ← auto-added at CreateAgent; owns an AudioSource
```

When you have **multiple speakers on one GameObject** (unusual), assign the correct `TryllSpeaker` to the `Speaker` slot on `TryllAgentComponent` in the Inspector, or set it from code before `CreateAgent` runs:

```
agentComp.Speaker = speakerForThisCharacter;
```

A graph with a `GenerateAndSpeak` or `Speak` node always gets a speaker, so its audio always plays. TTS audio is ignored only when the agent's graph has no TTS nodes and no speaker is assigned.

### Unreal

`UTryllAgentComponent` uses the `UTryllSpeakerComponent` on the same actor. If the actor has none and the graph contains a `GenerateAndSpeak` **or** `Speak` node, `CreateAgent` adds one — in the common case you don't need to do anything extra. Add a **Tryll Speaker** component yourself when something else needs it before the agent is created, such as Tryll Lip Sync:

```
Actor
  ├─ UTryllAgentComponent    ← graph with GenerateAndSpeak
  └─ UTryllSpeakerComponent  ← found or added at CreateAgent; manages hidden UAudioComponent
```

If the actor has **more than one** `UTryllSpeakerComponent`, set the **Speaker** property on `UTryllAgentComponent` explicitly in the Details panel (or from C++) to target the correct one.

## 5. React to playback events

### Unreal — `OnTtsAudioStarted` / `OnTtsAudioFinished`

`UTryllSpeakerComponent` fires two Blueprint-assignable delegates:

| Delegate             | When                                                                                                                                                                                                                                |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `OnTtsAudioStarted`  | First PCM chunk queued — speech is about to begin. Fires once per turn, even when several nodes speak.                                                                                                                              |
| `OnTtsAudioFinished` | The turn is complete and its queued audio has been handed to the audio engine — speech is ending. The engine's own buffers (a few tens of milliseconds) still play out. Also fires immediately when `StopPlayback` cuts speech off. |

Every `OnTtsAudioStarted` is followed by exactly one `OnTtsAudioFinished`. Use them to synchronize mouth-rig animations, idle transitions, or to gate the next `SendMessage` call:

```
// C++ — bind in BeginPlay after the Speaker is known.
Speaker->OnTtsAudioStarted.AddDynamic(this, &AMyNpc::OnSpeechStart);
Speaker->OnTtsAudioFinished.AddDynamic(this, &AMyNpc::OnSpeechEnd);
```

In Blueprint, drag from the **Tryll Speaker** component reference and select **Bind Event to On Tts Audio Started / Finished**.

To cut the NPC off (player interrupts, dialog skipped), call **Stop Playback** on the speaker. It drops the queued audio, fires `OnTtsAudioFinished`, and ignores further audio until the next speaking node starts. It does not stop generation — call `Cancel` on the agent too if the turn itself should end.

### Unity — end-of-audio detection

`TryllSpeaker` does not expose start/end events directly. Audio begins as soon as the first PCM chunk arrives and drains naturally. Two practical approaches:

- **Wait for `HasPendingAudio` to go false** after `OnTurnComplete` fires. No more audio arrives after `TurnComplete`, so false means the queue has drained down to the speaker's small (~100 ms) streaming buffer.
- **Use `OnTurnComplete` alone** if you only need to know when the LLM response is done (not when the last word finishes playing).

Don't use `AudioSource.isPlaying`: the speaker keeps its `AudioSource` playing between turns, so it never goes false.

```
agentComp.OnTurnComplete.AddListener((status, _, _) =>
{
    // LLM done; audio may still be playing.
    StartCoroutine(WaitForAudioEnd());
});

private IEnumerator WaitForAudioEnd()
{
    var speaker = GetComponent<TryllSpeaker>();
    yield return new WaitUntil(() => !speaker.HasPendingAudio);
    // Speech finished.
}
```

To cut the NPC off, call `speaker.StopPlayback()`.

## 6. Pick a language (multilingual voices)

Some TTS models speak many languages from one loaded model. **Supertonic 3** covers 31 (`en, de, fr, es, it, pt, pl, nl, ru, uk, ja, ko, …`). The voice profile's `Lang` field selects the synthesis language:

On the **Voice Profile** asset, set **Lang** to the language code (e.g. `de`).

On the **Tryll Voice Profile** asset, set **Lang** to the language code (e.g. `de`).

```
VoiceProfile(tts_model_name="Supertonic 3 (int8)", lang="de")  # speak German
```

Leave `Lang` empty to use the session locale, and then the model's catalog default (`en`). With `Lang` empty, a settings-menu language switch is just a session locale change (`SetSessionLocale`); the voice follows it from the next turn, with no agent rebuild. See [Ship in Another Language](https://docs.tryllengine.com/v0.9/how-to/ship-in-another-language.html).

Note

`Lang` selects how the model *pronounces* — your NPC still answers in whatever language the LLM generates. Steer the text language through the system prompt.

Models whose language is fixed per model (Pocket TTS, Orpheus, Maya1 — all English) **reject** a non-empty `Lang` with `3023 InvalidVoiceProfile`. With `Lang` empty and a session locale other than English, agent creation fails with `3017`: pick a voice that covers the language, or keep the session in English.

## 7. Pick or clone a voice

Which profile field selects the voice depends on the model family:

| Model               | Voice field        | How it works                                                                                                                                                                                                                                       |
| ------------------- | ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Supertonic 3        | `SpeakerId`        | 10 built-in voice styles — indices `0`–`9`.                                                                                                                                                                                                        |
| Pocket TTS          | `ReferenceVoice`   | Zero-shot cloning from a reference WAV (10–20 s recommended; the first 30 s are used); each agent can use a different clip on the same loaded model.                                                                                               |
| Qwen3-TTS 1.7B Base | `ReferenceVoice`   | Cloning from a reference WAV (3–10 s), required: there is no default voice. Speaks 10 languages from one clip; set `Lang`.                                                                                                                         |
| Orpheus 3B FT       | `SpeakerId`        | 8 named English speakers — indices `0`–`7` (Tara, Leah, Jess, Leo, Dan, Mia, Zac, Zoe). Inline tags such as `<laugh>` may appear in the spoken text.                                                                                               |
| Maya1 3B            | `VoiceDescription` | Natural-language identity (age, accent, pitch, timbre). Empty falls back to `SpeakerId` `0`–`1` (male / female 30s American). Steer delivery with [delivery modes](https://docs.tryllengine.com/v0.9/reference/voice-profile.html#delivery-modes). |

eSpeak-based TTS models are not supported

Model families that depend on eSpeak NG for text-to-phoneme conversion — **Piper/VITS (with espeak data), Kokoro, KittenTTS** — cannot be used with the Tryll server. (ZipVoice is not loadable either; it is rejected as an unknown family.) Supported Sherpa-ONNX families are **Supertonic** and **Pocket TTS**; a catalog entry with `tts_family` set to `vits`, `piper`, or `kokoro` fails to load with an error explaining this. llama.cpp SNAC families (**Orpheus**, **Maya1**) do not use eSpeak. See [TTS Models](https://docs.tryllengine.com/v0.9/reference/tts-models.html).

`SpeakerId` range is model-specific

`SpeakerId` must be below the model's speaker count — creating an agent with a larger value fails with `3023` and *"speaker_id N is out of range for TTS model … (M speaker(s); valid range 0..M-1)"*. Pocket TTS exposes exactly **one** speaker, so it must stay `0` there; change the Pocket voice with `ReferenceVoice`, not `SpeakerId`. Orpheus accepts `0`–`7`; Maya1 accepts `0`–`1` — `VoiceDescription` replaces the preset voice but does not widen the range.

For `ReferenceVoice` — cloning a brand-new voice from your own recording — see [Clone a Voice from an Audio Sample](https://docs.tryllengine.com/v0.9/how-to/clone-a-voice.html). To make a clip from a text description instead, see [Design a Voice Clip with Qwen3-TTS](https://docs.tryllengine.com/v0.9/how-to/design-a-voice-with-qwen3-tts.html), and for one clip per mood, [Add Emotions to a Voice](https://docs.tryllengine.com/v0.9/how-to/add-emotions-to-a-voice.html).

## 8. Change the voice or its delivery at runtime

Two calls change how an agent sounds after it is created. Neither needs the agent to be rebuilt.

- **Set a per-character default in the editor** with the agent component's **Voice Delivery** override. Its dropdown contains neutral and the assigned Voice profile's named modes. The override is used at creation without changing the shared profile asset; editing it on a live Unity or Unreal agent applies the new mode immediately.
- **Switch the delivery mode** with `SetAgentDeliveryMode(mode, intensity)`. The mode is one the profile defines (`angry`, `calm`, …) or the built-in `neutral`. It is accepted at any time, even mid-turn, and applies from the next sentence. What a mode can do depends on the model; see [Voice Profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html#delivery-modes).
- **Replace the whole voice** with `SetAgentVoice(profile)`, for example to slow a character down or move it to another model. It is accepted only while the agent is idle (not running, not paused); otherwise it fails with `3004 AgentBusy`. If the new profile is invalid, the old voice stays.

```
await agentComp.SetDeliveryModeAsync("angry", 0.8f);

var err = await agentComp.SetVoiceAsync(slowerVoiceProfile); // a TryllVoiceProfile
if (!err.IsOk)
    Debug.LogError($"SetVoice failed: {err}");
```

Call **Set Delivery Mode** (mode name and intensity) or **Set Voice** (a Tryll Voice Profile) on the agent component. Each reports completion through its delegate.

```
agent.set_delivery_mode("angry", intensity=0.8)
agent.set_voice(slower_voice)   # idle only
```

```
agent.SetDeliveryMode("angry", 0.8f);
agent.SetVoice(slowerVoice);    // idle only
```

`ChangeAgentParam` can no longer change any voice field. It still changes a speaking node's `min_sentence_chars` and `first_chunk_max_chars`; see [Change Agent Parameters at Runtime](https://docs.tryllengine.com/v0.9/how-to/change-agent-parameters.html).

## Write prompts for the voice

TTS speaks **exactly** what the language model writes, and speech models are trained on natural speech — not documents. Unconstrained LLM replies full of markdown (`**bold**`, `*smiles*` stage directions, bullet lists, headings, emoji) read terribly through any TTS, and for LM-based models like **Pocket TTS** they actively destabilize synthesis: expect repeated sentences, trailing babble, or random sounds after the sentence ends.

Give every voiced agent a system prompt that demands speakable text:

```
Reply with spoken words only: no markdown, no asterisks, no stage
directions, no lists, no emoji. Keep replies to one or two short sentences.
```

Further tips for clean audio:

- **Short sentences synthesize best** — they also stream sooner (lower first-audio latency). To start speaking before a long first sentence ends, set `FirstChunkMaxChars` (for example `48`): the first chunk may then end at a comma, or at the next space once it is that long, never under 5 words. It is off (`0`) by default — listen to it with your voice first, since a comma can end on a falling tone.
- **Spell out numbers that matter** ("twelve silver pieces", not "12 sp") — TTS models do minimal number expansion.
- Avoid content the model can't say: URLs, file paths, code identifiers.

The server automatically normalizes *characters* for Pocket TTS — typographic quotes/apostrophes (`’ “ ”`) to ASCII, ellipsis, parentheses, emoji removal, plus a terminal period, sentence casing, and short-input padding per Kyutai's reference pipeline. What it deliberately does **not** do is rewrite markup or structure: unspeakable text is a prompt problem, and silently rewriting it would only mask it. Fix it at the source with the format rules above.

## Common pitfalls

**No audio and no errors.** Check that a `TryllSpeaker` / `UTryllSpeakerComponent` is on the same actor, and that the graph actually contains a `GenerateAndSpeak` or `Speak` node (not a plain `Generate` node).

**Speak node produces no audio.** `Speak` voices its `input`-resolved slot, so it needs the node its `input` names (e.g. `Generate` or `CannedResponse`) to have written that slot **before** it runs on the same turn — remember that an empty `input` resolves to `user_message`, not the upstream node's output. A `Speak` node whose resolved slot was never written simply exits via `default` without synthesizing anything.

**Random sounds, repeated sentences, or trailing babble (Pocket TTS).** The synthesized text isn't speech-like — usually markdown, stage directions, or lists from an unconstrained LLM. Add the format rules from [Write prompts for the voice](#write-prompts-for-the-voice) to the agent's system prompt. The server log's `Pocket text prepared:` lines (debug level) show exactly what text entered the model.

**Agent creation fails with `3025 AgentVoiceRequired`.** The graph has a `Speak` or `GenerateAndSpeak` node but the agent was created without a voice. Assign a voice profile on the agent component (Unity **Voice**, Unreal **Voice Profile**), or pass `voice` when you create the agent from code. There is no default voice.

**Agent creation fails with `3023` "speaker_id N is out of range".** The selected TTS model has fewer speakers than the `SpeakerId` you set. The shipped Supertonic 3 bundle accepts `0`–`9`; Pocket TTS only `0` (select its voice with `ReferenceVoice` instead); Orpheus `0`–`7`; Maya1 `0`–`1` (`VoiceDescription` does not widen it). The error message states the model's valid range.

**Agent creation fails with `3023` "voice: TTS model '…': …".** The profile's `TtsModelName` is empty, is not in `models.json`, or the model could not be loaded. List the valid names (Unity `RequestListModelsAsync()`, Unreal `RequestListModels`, C++ `ListModels()`, Python `list_models()`), or acquire the model via the editor Model Manager. Other `3023` messages name the profile field the model cannot use, for example a `ReferenceVoice` on Supertonic 3.

**Audio plays in bursts or drops mid-sentence (Unity).** The `TryllSpeaker` chunk queue (1024 chunks) overflowed — TTS synthesis outran realtime playback, which can happen with very long responses; the newest chunks are dropped and a warning is logged. Keep replies shorter or reduce the LLM generation budget (`max_tokens`). Lowering `MinSentenceChars` makes more, smaller chunks, so it overflows sooner. The Unreal speaker has no such cap.

**`OnTtsAudioFinished` never fires (Unreal).** It fires only after the turn's `TurnComplete` and once the audio engine has taken all queued audio. Check that you bound it on the speaker the agent actually uses — with several speakers on the actor, that is the agent's **Speaker** property. Without an audio device (for example `-nosound`), nothing consumes the queue, so it never fires.

## Related

- [Concept: TTS and Voice Output](https://docs.tryllengine.com/v0.9/concepts/tts-and-voice-output.html)
- [Reference: TTS Models](https://docs.tryllengine.com/v0.9/reference/tts-models.html) — the supported voice families and why the rest are unavailable
- [Reference: Voice Profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html) — every voice field, delivery modes, and what each model can do with them
- [Reference: Generate and Speak node](https://docs.tryllengine.com/v0.9/reference/nodes/generate-and-speak.html)
- [Reference: TryllSpeaker (Unity)](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-speaker.html)
- [Reference: UTryllSpeakerComponent (Unreal)](https://docs.tryllengine.com/v0.9/reference/client-api/unreal/cpp/index.html)
- [How-to: Add Lip Sync to a Character](https://docs.tryllengine.com/v0.9/how-to/add-lip-sync-to-a-character.html) — move the character's mouth with the voice
- [How-to: Clone a Voice from an Audio Sample](https://docs.tryllengine.com/v0.9/how-to/clone-a-voice.html)
- [How-to: Add Emotions to a Voice](https://docs.tryllengine.com/v0.9/how-to/add-emotions-to-a-voice.html) — one clip per mood, switched with delivery modes
- [How-to: Change Agent Parameters at Runtime](https://docs.tryllengine.com/v0.9/how-to/change-agent-parameters.html)
- [How-to: Use Voice Input](https://docs.tryllengine.com/v0.9/how-to/use-voice-input.html) — add microphone input alongside TTS output
