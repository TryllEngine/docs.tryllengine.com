# Add Lip Sync to a Character

Make a speaking character's mouth move with its voice. The server sends a **viseme track** (mouth shapes with timings) alongside the speech, and a lip sync component plays it on the character's face in step with the audio: `TryllLipSync` in Unity, **Tryll Lip Sync** in Unreal. Nothing is precomputed and no extra model runs.

For a MetaHuman, use [Add Lip Sync to a MetaHuman](https://docs.tryllengine.com/v0.9/how-to/add-lip-sync-to-metahuman.html) with its Viseme solver instead.

For how the track is produced and what it can and cannot do, see [Concept: TTS and Voice Output](https://docs.tryllengine.com/v0.9/concepts/tts-and-voice-output.html#lip-sync-visemes).

Prerequisites

- A character that already speaks, with a [voice profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html) and a [GenerateAndSpeak](https://docs.tryllengine.com/v0.9/reference/nodes/generate-and-speak.html) or [Speak](https://docs.tryllengine.com/v0.9/reference/nodes/speak.html) node — see [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.9/how-to/add-voice-to-an-agent.html).
- A face mesh with **ARKit blendshapes** (morph targets in Unreal): `jawOpen`, `mouthClose`, `mouthFunnel`, `mouthPucker`, `mouthStretchLeft`, …. Most avatar pipelines can export them. Unity needs the names to match exactly; Unreal ignores case.

## 1. Turn on visemes in the voice profile

Set `emit_visemes` on the agent's voice profile. It is off by default. A lip-synced character either has visemes or not, so the flag is on the voice, not on a node: with it on, every speaking node of the agent, `GenerateAndSpeak` and `Speak` alike, sends a viseme track.

Select the character's **Voice Profile** asset and tick **Emit Visemes**.

Open the character's **Tryll Voice Profile** asset and tick **Emit Visemes**.

```
VoiceProfile(
    tts_model_name='Supertonic 3 (int8)',
    emit_visemes=True,
)
```

To turn visemes on or off for a live agent, replace its voice with `SetAgentVoice` while the agent is idle.

## 2. Add TryllLipSync to the character (Unity)

1. Select the GameObject that has your `TryllAgentComponent`.
1. **Add Component → Tryll → Tryll Lip Sync**. It needs a `TryllSpeaker` on the same GameObject — the speaker is both the audio and the clock the face follows — and adds one if it is missing.
1. Drag that `TryllSpeaker` into the **Speaker** field of the `TryllAgentComponent`, so the agent plays its audio through the speaker the face follows.
1. Set **Face Root** to the parent of the face meshes (for example the imported avatar's root). The component finds every `SkinnedMeshRenderer` under it. To bind specific meshes only, fill **Renderers** instead. If the face is spawned at runtime, call `BindFace(avatarRoot)` once it exists.
1. Enter Play mode and make the character speak. The Console logs the blendshape weight scale it found on the mesh (`1` for glTF imports, `100` for FBX imports — both are handled).

Only the mouth is driven

`TryllLipSync` writes only the blendshapes its viseme map uses (the mouth and jaw channels). Blinks, brows and expressions on the same mesh are left alone, so other animation can drive them. Channels you add to the **Map** become owned by lip sync.

## 2. Add Tryll Lip Sync to the character (Unreal)

1. Open the character's Blueprint. It needs a **Tryll Speaker** component next to its **Tryll Agent**, as in [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.9/how-to/add-voice-to-an-agent.html). The speaker is both the audio and the clock the face follows. With one speaker on the actor, the agent and the lip sync both find it by themselves.
1. **Add Component → Tryll Lip Sync**.
1. Press **Play** and make the character speak.

When the character starts, the component uses the Tryll Speaker on the same actor and looks through the actor's skeletal meshes for one with ARKit morph targets. If several have them, a mesh whose name contains `Face` wins. To choose yourself, set **Speaker** and **Face Mesh**. If the face is spawned at runtime, call **Bind Face** once it exists. **Is Face Bound** reports whether any ARKit morph target was found.

Only the mouth is driven

**Tryll Lip Sync** writes only the morph targets its pose table uses. The rest of the face is left to the character's animation. While a mouth morph target is non-zero it overrides the AnimBP's value for it, and at rest the AnimBP's value applies again.

## 3. Tune the look

All settings are on the component and can be changed in Play mode, except **Map** (read when the component wakes) and **Compensate Output Latency** (read at start).

| Setting                          | Default  | What it does                                                                                                                       |
| -------------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| **Intensity**                    | `0.4`    | Scales every mouth pose. Raise it for a more pronounced mouth, lower it if speech looks exaggerated. `0` keeps the mouth at rest.  |
| **Smoothing Per Second**         | `16`     | How quickly the face moves toward each new shape. Lower is softer, higher is snappier.                                             |
| **Closure Smoothing Per Second** | `60`     | Faster rate used for lip closures (p, b, m, f, v) so the lips actually meet on short sounds.                                       |
| **Attack Decay Ms**              | `50`     | Cross-fade between consecutive shapes.                                                                                             |
| **Max Hold Ms**                  | `400`    | Returns the face to rest if a turn is cancelled mid-sentence.                                                                      |
| **Map**                          | built in | The pose (ARKit channel weights) used for each of the 15 visemes. Leave it empty to use the built-in table; edit it to suit a rig. |

Changing the smoothing does not shift sync: with **Compensate Smoothing Lag** on (the default), the component looks ahead by the delay the smoothing adds.

The **Intensity** is also available from code:

```
GetComponent<TryllLipSync>().Intensity = 0.6f;
```

In Unreal, the same settings are on the **Tryll Lip Sync** component, with the same defaults, and can be changed during Play except **Poses**. The differences:

- The pose table is called **Poses** and is read at BeginPlay.
- There is no **Output Lead Trim Ms**: Unreal needs no correction for a streaming-clip backlog. Both engines have **Compensate Output Latency** (on by default); see [Check the sync](#4-check-the-sync).
- From Blueprint, set the intensity with **Set Intensity**.

## 4. Check the sync

The mouth should neither trail nor lead the voice. If it consistently does:

- **Sync Offset Ms** shifts the face against the audio. Positive values move the face earlier.
- **Output Lead Trim Ms** (default `342`) accounts for how far Unity's streaming audio buffer runs ahead of what is audible. It was measured on a typical Windows setup; a different audio device or DSP buffer size can change it. Adjust it if the offset you need is large. Unity only.
- **Compensate Output Latency** (on by default, both engines) holds the face back by the audio engine's output buffer, read from the audio device's buffer settings, so the mouth lines up with the sound as heard. In Unreal, the Tryll Speaker's **Get Output Latency Ms** returns the value in use; in Unity, `TryllLipSync.OutputLatencyMs`.

## Use the track directly

If you drive a face some other way, subscribe to the raw events instead of using `TryllLipSync`. Each frame covers one synthesized sentence. Offsets are in samples from the start of the turn's audio (its one format announcement), and keep counting across every speaking node of the turn, at the sample rate from the audio format callback. Each shape lasts until the next event; `sil` is rest.

```
agentComponent.Agent.OnTtsVisemes += (nodeName, events) =>
{
    foreach (var e in events)
        Debug.Log($"{e.SampleOffset}: {e.Viseme} ({e.Weight:F2})");
};
```

```
Speaker->OnTtsVisemes.AddLambda(
    [](const FString& NodeName, TArrayView<const FTryllVisemeEvent> Events)
{
    for (const FTryllVisemeEvent& E : Events)
    {
        UE_LOG(LogTemp, Log, TEXT("%lld: %s (%.2f)"), E.SampleOffset,
               *UEnum::GetValueAsString(E.Viseme), E.Weight);
    }
});
```

`Speaker` is the character's `UTryllSpeakerComponent`. The view is valid only during the callback.

```
def on_visemes(node_name, events):
    for e in events:
        print(e.sample_offset, e.viseme, e.weight)

agent.set_on_tts_visemes(on_visemes)
```

```
agent.SetOnTtsVisemes([](std::string_view nodeName,
                         std::span<const Tryll::Client::AgentProxy::VisemeEvent> events)
{
    for (const auto& e : events)
        std::printf("%u: %d (%.2f)\n", e.sampleOffset, int(e.viseme), e.weight);
});
```

`weight` (0–1) is how far the shape should open, from the loudness of that stretch of speech. Lip closures and rests always have weight `1`.

## Troubleshooting

| Symptom                       | Check                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| The mouth never moves         | **Emit Visemes** is on in the agent's voice profile; `TryllLipSync` has the agent assigned (a Console warning says so if not); the agent's **Speaker** is the `TryllSpeaker` next to `TryllLipSync`; the face mesh has ARKit-named blendshapes.                                                                                                                                                                                                                                |
| The mouth moves too much      | Lower **Intensity**.                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| The mouth pops between shapes | Lower **Smoothing Per Second** or raise **Attack Decay Ms**.                                                                                                                                                                                                                                                                                                                                                                                                                   |
| Mouth lags or leads the voice | Adjust **Sync Offset Ms**; for a large offset, re-check **Output Lead Trim Ms** (Unity) or that **Compensate Output Latency** is on (Unreal).                                                                                                                                                                                                                                                                                                                                  |
| Unreal: the mouth never moves | The Output Log (`LogTryll`) says whether **Tryll Lip Sync** found no Tryll Speaker or no face mesh with ARKit morph targets. Also check **Emit Visemes** is on in the agent's voice profile. If the actor has several Tryll Speakers, both components pick the first one found; to use another, set it as **Speaker** on both — on the placed instance, or from Blueprint before the agent is created (the agent's **Speaker** cannot be set in the Blueprint class defaults). |

## See also

- [Add Lip Sync to a MetaHuman (Unreal)](https://docs.tryllengine.com/v0.9/how-to/add-lip-sync-to-metahuman.html)
- [Reference: TryllLipSync (Unity)](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-lip-sync.html)
- [Concept: TTS and Voice Output](https://docs.tryllengine.com/v0.9/concepts/tts-and-voice-output.html)
- [Reference: Voice Profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html)
- [Reference: GenerateAndSpeak node](https://docs.tryllengine.com/v0.9/reference/nodes/generate-and-speak.html)
- [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.9/how-to/add-voice-to-an-agent.html)
