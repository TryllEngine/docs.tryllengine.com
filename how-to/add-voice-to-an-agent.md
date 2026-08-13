# Add Voice Output to an Agent

Make an NPC speak. This guide covers every step: configuring the session with a TTS engine, building a graph that produces speech — with the fused [GenerateAndSpeak](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html) node or the TTS-only [Speak](https://docs.tryllengine.com/reference/nodes/speak.html) node — wiring up the playback component, and reacting to audio lifecycle events.

Use **GenerateAndSpeak** to speak a generated answer with the lowest latency (synthesis overlaps generation). Use **Speak** to voice text that did *not* come from a generation pass — e.g. a scripted [CannedResponse](https://docs.tryllengine.com/reference/nodes/canned-response.html) — or any upstream text you want spoken without a second LLM cost. Both are wired to the client exactly the same way; only the graph differs.

For background on how streaming TTS works, see [Concept: TTS and Voice Output](https://docs.tryllengine.com/concepts/tts-and-voice-output.html).

Prerequisites

- A connected session — see [Connect and Manage a Session](https://docs.tryllengine.com/how-to/connect-and-manage-session.html).
- A TTS model available on the server. Acquire it beforehand — via the editor Model Manager or [Use Your Own Local Model](https://docs.tryllengine.com/how-to/use-your-own-local-model.html) — since `CreateAgent` fails fast on a missing model rather than downloading it for you.

## 1. Create the session with a TTS engine

Pass a TTS engine when you create the session. This is a one-time call made immediately after the connection is established.

```
client.CreateSession(
    TryllInferenceEngine.LlamaCpp,
    ttsEngine: TryllInferenceEngine.SherpaOnnx);
```

```
Subsystem->CreateSession(
    ETryllInferenceEngine::LlamaCpp,
    /*GameName=*/ TEXT(""),
    /*SttEngine=*/ ETryllInferenceEngine::Mock,
    /*TtsEngine=*/ ETryllInferenceEngine::SherpaOnnx);
```

Or from Blueprint: in your `OnConnectionChanged(true)` handler, call **Create Session** and set **Tts Engine** to `SherpaOnnx`.

Warning

If `TtsEngine` is omitted (or left at its default, `Mock`), the server has no engine to synthesize speech. `GenerateAndSpeak` will still produce text, but no `TtsAudioFrame` messages will be emitted.

## 2. Build a graph with a GenerateAndSpeak node

Replace any `Generate` node in your graph with `GenerateAndSpeak`. All extra fields are optional: leave `TtsModelName` empty and the server uses the first TTS voice in its catalog, or set it to pick a specific voice. `SpeakerId`, `Speed`, `TtsLang` (synthesis language on multilingual models), and `TtsVoice` (reference clip for voice-cloning models) are optional and can be changed at runtime.

1. In the Project window: right-click → **Create → Tryll → Workflow Asset**.
1. Under **Graph → Nodes**, add a node named `speak` and pick **GenerateAndSpeak** in the **Params** type picker.
1. Set **Tts Model Name** from the dropdown (e.g. `Supertonic 3 (int8)`). Only registered TTS models are listed — register it first in the [Model Manager](https://docs.tryllengine.com/how-to/manage-models-in-editor.html) if it's missing.
1. Set **Speaker Id** and **Speed** as needed; leave **Default Exit** empty (END) unless you chain further nodes.
1. Set **Start Node** to `speak` and **Default Model Name** to your language model, then assign the asset to your `TryllAgentComponent`'s **Workflow Asset** field.

The same graph built in code, for agents you create directly rather than through a `TryllAgentComponent`:

```
var graph = new TryllGraphBuilder()
    .AddGenerateAndSpeak("speak", new TryllGenerateAndSpeakParams
    {
        TtsModelName     = "Supertonic 3 (int8)",  // must match a name in models.json
        SpeakerId        = 0,                // voice index (0 = default)
        Speed            = 1.0f,
        DefaultExit      = "",               // END
    })
    .SetStartNode("speak")
    .SetDefaultModelName("My Language Model")
    .Build();

var (agent, error) = await TryllClient.Instance.RequestCreateAgentAsync(graph);
```

1. Open or create a **Tryll Workflow Asset** in the Content Browser.
1. Add a **GenerateAndSpeak** node.
1. In the Details panel, enable **Override Tts Model Name** and pick the model from the **dropdown** (e.g. `Supertonic 3 (int8)`). Only registered TTS models are listed — register it first in the [Model Manager](https://docs.tryllengine.com/how-to/manage-models-in-editor.html) if it's missing.
1. Set **Speaker Id** and **Speed** as needed.
1. Leave **Default Exit** empty (END) unless you chain further nodes.
1. Set **Start Node** to `speak` and assign the asset to your `UTryllAgentComponent`.

```
auto* SpeakParams = NewObject<UTryllGenerateAndSpeakParams>(this);
SpeakParams->bOverrideTtsModelName = true;
SpeakParams->TtsModelName          = TEXT("Supertonic 3 (int8)");
SpeakParams->SpeakerId             = 0;
SpeakParams->Speed                 = 1.0f;

FTryllGraphDescription Graph;
FTryllNodeDescription Node;
Node.Name   = TEXT("speak");
Node.Params = SpeakParams;
Graph.Nodes.Add(Node);
Graph.StartNode           = TEXT("speak");
Graph.DefaultModelName    = TEXT("My Language Model");

// Create the agent directly from the graph. To drive a
// UTryllAgentComponent instead, author the same graph as a
// UTryllWorkflowAsset and assign it to WorkflowAsset.
Subsystem->RequestCreateAgent(Graph,
    [](TSharedPtr<FTryllAgent> Agent, FTryllError Error) { /* … */ });
```

### Alternative: voice scripted text with a Speak node

When the text to speak does **not** come from the language model — e.g. a fixed [CannedResponse](https://docs.tryllengine.com/reference/nodes/canned-response.html) — chain a [Speak](https://docs.tryllengine.com/reference/nodes/speak.html) node after the node that produced it. `Speak` voices its `input`-resolved slot and runs no LLM inference, so a scripted line costs nothing to generate — set `input` to the upstream node's output slot name (empty `input` defaults to `user_message`, like every other input-bearing node, so an explicit `input` is required here). `TtsModelName`, `SpeakerId`, `Speed`, and `MinSentenceChars` behave exactly as on `GenerateAndSpeak` (and `TtsModelName` may likewise be left empty to use the first catalog voice).

```
var graph = new TryllGraphBuilder()
    .AddCannedResponse("line", new TryllCannedResponseParams
    {
        // … canned-response config …
        DefaultExit = "say",          // route into the Speak node
    })
    .AddSpeak("say", new TryllSpeakParams
    {
        Input        = "line",        // voice the CannedResponse "line" slot
        TtsModelName = "",            // empty → first available voice
        SpeakerId    = 0,
        Speed        = 1.0f,
        DefaultExit  = "",            // END
    })
    .SetStartNode("line")
    .Build();
```

```
auto* SpeakParams = NewObject<UTryllSpeakParams>(this);
SpeakParams->bOverrideInput        = true;
SpeakParams->Input                 = FTryllSlotInput(TEXT("line")); // voice the CannedResponse "line" slot
SpeakParams->bOverrideTtsModelName = false;   // unset → first available voice
SpeakParams->SpeakerId             = 0;
SpeakParams->Speed                 = 1.0f;
// … add a CannedResponse node named "line" routing its DefaultExit to "say",
//    then add SpeakParams as node "say" …
```

1. Add your text-producing node (e.g. **CannedResponse**) and a **Speak** node.
1. Set the producer node's **Default Exit** to the Speak node's name.
1. On the Speak node, enable **Override Input** and set **Input** to the producer node's output slot (its node name, e.g. `line`). Without this, an empty **Input** voices `user_message` instead of the scripted line.
1. Optionally enable **Override Tts Model Name** to pick a specific voice; leave it off to use the first catalog voice.

Latency

`Speak` only sees text after the upstream node has fully finished, so chained after a `Generate` node audio begins only once generation completes. To speak a *generated* answer with minimal delay, prefer `GenerateAndSpeak`, which overlaps synthesis with generation. See [Speak](https://docs.tryllengine.com/reference/nodes/speak.html) and [Generate and Speak](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html).

## 3. Add the speaker component

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

To **suppress TTS playback** entirely, assign no speaker and remove any `TryllSpeaker` component from the GameObject.

### Unreal

Add a **Tryll Speaker** component (`UTryllSpeakerComponent`) to the same actor as your `UTryllAgentComponent`. The agent component auto-discovers the first `UTryllSpeakerComponent` on the actor at `BeginPlay`:

```
Actor
  ├─ UTryllAgentComponent    ← graph with GenerateAndSpeak
  └─ UTryllSpeakerComponent  ← auto-discovered; manages hidden UAudioComponent
```

If the actor has **more than one** `UTryllSpeakerComponent`, set the **Speaker** property on `UTryllAgentComponent` explicitly in the Details panel (or from C++) to target the correct one.

## 4. React to playback events

### Unreal — `OnTtsAudioStarted` / `OnTtsAudioFinished`

`UTryllSpeakerComponent` fires two Blueprint-assignable delegates:

| Delegate             | When                                               |
| -------------------- | -------------------------------------------------- |
| `OnTtsAudioStarted`  | First PCM chunk queued — speech is about to begin. |
| `OnTtsAudioFinished` | Audio component drained — speech is complete.      |

Use them to synchronize mouth-rig animations, idle transitions, or to gate the next `SendMessage` call:

```
// C++ — bind in BeginPlay after the Speaker is known.
Speaker->OnTtsAudioStarted.AddDynamic(this, &AMyNpc::OnSpeechStart);
Speaker->OnTtsAudioFinished.AddDynamic(this, &AMyNpc::OnSpeechEnd);
```

In Blueprint, drag from the **Tryll Speaker** component reference and select **Bind Event to On Tts Audio Started / Finished**.

### Unity — end-of-audio detection

`TryllSpeaker` does not expose start/end events directly. Audio begins as soon as the first PCM chunk arrives and drains naturally. Two practical approaches:

- **Poll `AudioSource.isPlaying`** in a coroutine after `OnTurnComplete` fires to know when the queue has fully drained.
- **Use `OnTurnComplete` alone** if you only need to know when the LLM response is done (not when the last word finishes playing).

```
agentComp.OnTurnComplete.AddListener((status, _, _) =>
{
    // LLM done; audio may still be playing.
    StartCoroutine(WaitForAudioEnd());
});

private IEnumerator WaitForAudioEnd()
{
    var src = GetComponent<TryllSpeaker>()
                  .GetComponent<AudioSource>();
    yield return new WaitUntil(() => !src.isPlaying);
    // Speech finished.
}
```

## 5. Pick a language (multilingual voices)

Some TTS models speak many languages from one loaded model. **Supertonic 3** covers 31 (`en, de, fr, es, it, pt, pl, nl, ru, uk, ja, ko, …`); the `TtsLang` parameter selects the synthesis language per agent:

```
.AddGenerateAndSpeak("speak", new TryllGenerateAndSpeakParams
{
    TtsModelName = "Supertonic 3 (int8)",
    TtsLang      = "de",          // speak German
    DefaultExit  = "",
})
```

On the **GenerateAndSpeak** node, enable **Override Tts Lang** and enter the language code (e.g. `de`).

Leave `TtsLang` empty to use the model's catalog default (`en`). An unsupported code fails the turn's synthesis with a server-side error listing the valid codes. `TtsLang` is mutable, so a settings-menu language switch is just a [parameter change](https://docs.tryllengine.com/how-to/change-agent-parameters.html) — no agent rebuild.

Note

`TtsLang` selects how the model *pronounces* — your NPC still answers in whatever language the LLM generates. Steer the text language through the system prompt.

Models whose language is fixed per model (e.g. Pocket TTS) ignore `TtsLang`.

## 6. Pick or clone a voice

Which knob selects the voice depends on the model family:

| Model        | Voice knob  | How it works                                                                                                  |
| ------------ | ----------- | ------------------------------------------------------------------------------------------------------------- |
| Supertonic 3 | `SpeakerId` | 10 built-in voice styles — indices `0`–`9`.                                                                   |
| Pocket TTS   | `TtsVoice`  | Zero-shot cloning from a 10–20 s reference WAV; each agent can use a different clip on the same loaded model. |

eSpeak-based TTS models are not supported

Model families that depend on eSpeak NG for text-to-phoneme conversion — **Piper/VITS (with espeak data), Kokoro, KittenTTS, ZipVoice** — cannot be used with the Tryll server. Supported TTS families are **Supertonic** and **Pocket TTS**; a catalog entry with `tts_family` set to `vits`, `piper`, or `kokoro` fails to load with an error explaining this.

`SpeakerId` range is model-specific

`SpeakerId` must be below the model's speaker count — creating an agent with a larger value fails with *"speaker_id N is out of range for TTS model … (M speaker(s); valid range 0..M-1)"*. Pocket TTS exposes exactly **one** speaker, so it must stay `0` there; change the Pocket voice with `TtsVoice`, not `SpeakerId`.

For `TtsVoice` — including cloning a brand-new voice from your own recording — see [Clone a Voice from an Audio Sample](https://docs.tryllengine.com/how-to/clone-a-voice.html).

## 7. Adjust speed and voice at runtime

`speed`, `speaker_id`, `tts_lang`, and `tts_voice` are mutable — change them between turns without recreating the agent. See [Change Agent Parameters at Runtime](https://docs.tryllengine.com/how-to/change-agent-parameters.html) for the full API; a TTS-specific example:

```
// GetNodeParamsBaseline already returns a deep copy — mutate in place, send.
var p = (TryllGenerateAndSpeakParams)agentComp.GetNodeParamsBaseline("speak");
p.Speed = 0.8f;   // slow to 80 %
agentComp.ChangeParams("speak", p, err =>
{
    if (!err.IsOk)
        Debug.LogError($"ChangeParams failed: {err}");
});
```

```
auto* Baseline = Cast<UTryllGenerateAndSpeakParams>(
    AgentComp->GetNodeParamsBaseline(TEXT("speak")));
auto* Updated = Cast<UTryllGenerateAndSpeakParams>(
    UTryllNodeParamsFactory::CloneParams(Baseline, this));
Updated->Speed = 0.8f;
AgentComp->ChangeParams(TEXT("speak"), Updated);
```

1. Get the agent component reference.
1. Call **Get Node Params Baseline** with `"speak"`.
1. Cast the result to **UTryll Generate And Speak Params**.
1. Call **Clone Params** (from `UTryllNodeParamsFactory`).
1. Set **Speed** on the clone.
1. Call **Change Params** on the agent component.

Note

`TtsModelName` is structural — it cannot be changed at runtime. To use a different TTS model, create a new agent.

## Write prompts for the voice

TTS speaks **exactly** what the language model writes, and speech models are trained on natural speech — not documents. Unconstrained LLM replies full of markdown (`**bold**`, `*smiles*` stage directions, bullet lists, headings, emoji) read terribly through any TTS, and for LM-based models like **Pocket TTS** they actively destabilize synthesis: expect repeated sentences, trailing babble, or random sounds after the sentence ends.

Give every voiced agent a system prompt that demands speakable text:

```
Reply with spoken words only: no markdown, no asterisks, no stage
directions, no lists, no emoji. Keep replies to one or two short sentences.
```

Further tips for clean audio:

- **Short sentences synthesize best** — they also stream sooner (lower first-audio latency).
- **Spell out numbers that matter** ("twelve silver pieces", not "12 sp") — TTS models do minimal number expansion.
- Avoid content the model can't say: URLs, file paths, code identifiers.

The server automatically normalizes *characters* for Pocket TTS — typographic quotes/apostrophes (`’ “ ”`) to ASCII, ellipsis, parentheses, emoji removal, plus a terminal period, sentence casing, and short-input padding per Kyutai's reference pipeline. What it deliberately does **not** do is rewrite markup or structure: unspeakable text is a prompt problem, and silently rewriting it would only mask it. Fix it at the source with the format rules above.

## Common pitfalls

**No audio and no errors.** Check that a `TryllSpeaker` / `UTryllSpeakerComponent` is on the same actor, and that the graph actually contains a `GenerateAndSpeak` or `Speak` node (not a plain `Generate` node).

**Speak node produces no audio.** `Speak` voices its `input`-resolved slot, so it needs the node its `input` names (e.g. `Generate` or `CannedResponse`) to have written that slot **before** it runs on the same turn — remember that an empty `input` resolves to `user_message`, not the upstream node's output. A `Speak` node whose resolved slot was never written simply exits via `default` without synthesizing anything.

**Random sounds, repeated sentences, or trailing babble (Pocket TTS).** The synthesized text isn't speech-like — usually markdown, stage directions, or lists from an unconstrained LLM. Add the format rules from [Write prompts for the voice](#write-prompts-for-the-voice) to the agent's system prompt. The server log's `Pocket text prepared:` lines (debug level) show exactly what text entered the model.

**Agent creation fails with "speaker_id N is out of range".** The selected TTS model has fewer speakers than the `SpeakerId` you set. The shipped Supertonic 3 bundle accepts `0`–`9`; Pocket TTS only `0` (select its voice with `TtsVoice` instead). The error message states the model's valid range.

**Error 4002 "TTS model not found".** The `TtsModelName` does not match any entry in `models.json`. Leave it empty to use the first available catalog voice, run `client.ListModels()` to see valid names, or acquire the model via the editor Model Manager.

**Audio plays in bursts or drops mid-sentence.** The chunk queue overflowed — TTS synthesis outran realtime playback (this can happen with very long responses). Lower `MinSentenceChars` to produce smaller, more frequent audio chunks, or reduce the LLM generation budget.

**`OnTtsAudioFinished` fires before audio ends (Unreal).** This happens when `UTryllSpeakerComponent` is added to the actor after `BeginPlay` — the hidden `UAudioComponent` was not yet created. Ensure the component is present before `BeginPlay` runs (add it in the actor's Blueprint or C++ constructor, not at runtime).

## Related

- [Concept: TTS and Voice Output](https://docs.tryllengine.com/concepts/tts-and-voice-output.html)
- [Reference: TTS Models](https://docs.tryllengine.com/reference/tts-models.html) — the supported voice families and why the rest are unavailable
- [Reference: Generate and Speak node](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html)
- [Reference: TryllSpeaker (Unity)](https://docs.tryllengine.com/reference/client-api/unity/tryll-speaker.html)
- [Reference: UTryllSpeakerComponent (Unreal)](https://docs.tryllengine.com/reference/client-api/unreal/cpp/index.html)
- [How-to: Clone a Voice from an Audio Sample](https://docs.tryllengine.com/how-to/clone-a-voice.html)
- [How-to: Change Agent Parameters at Runtime](https://docs.tryllengine.com/how-to/change-agent-parameters.html)
- [How-to: Use Voice Input](https://docs.tryllengine.com/how-to/use-voice-input.html) — add microphone input alongside TTS output
