# Add Emotions to a Voice with Delivery Modes

Make a character sound angry, afraid or joyful, and switch between moods while the game runs. A voice profile holds one voice and a list of **delivery modes**: named ways of speaking, such as `angry` or `whisper`. Game code switches the mode with `SetAgentDeliveryMode`, and the next sentence is spoken in the new mood.

With a **cloning** model (Qwen3-TTS Base or Pocket TTS), each mood is a **reference clip of the same character speaking in that mood**. This page shows how to make those clips, put them in a profile, and switch between them.

Prerequisites

- Voice output working — see [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.9/how-to/add-voice-to-an-agent.html).
- A cloning model, `Qwen3-TTS 12Hz 1.7B Base (Q4_K_M)` or `Pocket TTS (int8)`, and a neutral reference clip for the character — see [Clone a Voice from an Audio Sample](https://docs.tryllengine.com/v0.9/how-to/clone-a-voice.html).

## Why one clip per mood

Cloning copies the **register and energy** of the reference clip along with the voice. In our measurements, neutral text cloned from a shouted, angry clip came out raised and loud on both models. Cloned from a sad clip, it came out low and flat. Cloning models have no emotion parameter: setting a delivery mode's **Instruction** on them fails with `3023`. So the clip *is* the mood:

1. One clip per character per mood, for example `voices/guard/neutral.wav`, `voices/guard/angry.wav`, `voices/guard/afraid.wav`.
1. The neutral clip in the profile's **Reference Voice**. That is the built-in `neutral` mode.
1. One delivery mode per mood in the same profile, each with its own **Reference Voice**.
1. `SetAgentDeliveryMode("angry")` at runtime. It needs no agent rebuild.

**Every mood must sound like the same person.** A clip from another speaker changes who is talking, not just how.

## 1. Plan the moods

- **Pick the few moods your game uses.** Each one is a clip to make and to check. Three to five per character is plenty for most NPCs.
- **Use the same mode names on every character.** Then one call, such as `SetAgentDeliveryMode("angry")`, works on everyone. A mode with no fields set sounds like `neutral`, so a character without an angry clip can still define an empty `angry` mode.
- **Keep names short and lowercase**, such as `angry`, `afraid`, `joyful`. Game code and the editor's dropdown use them as written.

## 2. Make one clip per mood

There are two ways to get a set of clips that sound like one person.

### Record one actor

This gives the best result. Record every mood with the same actor, in the same session, with the same microphone and room. Follow the clip rules in [Clone a Voice → Record a reference clip](https://docs.tryllengine.com/v0.9/how-to/clone-a-voice.html#1-record-or-choose-a-reference-clip). Ask for each mood in character, and keep the same distance to the microphone, even when the actor shouts or whispers.

### Design the moods with Qwen3-TTS

Without an actor, render each mood with Qwen3-TTS **VoiceDesign**. The steps for one clip are in [Design a Voice Clip with Qwen3-TTS](https://docs.tryllengine.com/v0.9/how-to/design-a-voice-with-qwen3-tts.html). For a set of moods:

1. **Split the description into identity and delivery.** The identity (gender, age, pitch, timbre, accent) stays **word for word** in every mood. Only the delivery changes.
1. **Write a line in each mood.** An angry line for the angry clip, a frightened line for the afraid one.
1. **Render several takes per mood** and keep the ones that sound most like the neutral clip. Listen to them side by side. Drop a take that sounds like someone else, even if its mood is right.

For example, a palace guard:

| Mood      | Voice description                                                                                                                                                      |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `neutral` | A man in his forties with a deep, slightly hoarse voice and a light Northern English accent. **He speaks calmly, at an even pace.**                                    |
| `angry`   | A man in his forties with a deep, slightly hoarse voice and a light Northern English accent. **He is furious and shouts, fast and sharp.**                             |
| `afraid`  | A man in his forties with a deep, slightly hoarse voice and a light Northern English accent. **He is frightened; his voice shakes and he speaks quickly and quietly.** |

Each render is a new sample, so designed moods can drift slightly away from each other. Picking takes by ear keeps the drift small.

Put the clips together in one folder per character:

```
<storage root>/
  └─ voices/
      └─ guard/
          ├─ neutral.wav
          ├─ angry.wav
          └─ afraid.wav
```

## 3. Add the modes to the voice profile

1. Select the character's **Voice Profile** asset (**Create → Tryll → Voice Profile** if it has none).

1. Set **Tts Model Name** to the cloning model and **Reference Voice** to the neutral clip.

1. Under **Delivery Modes**, add one element per mood. Set its **Name** (`angry`) and its **Reference Voice** (`voices/guard/angry.wav`, with the **…** button or typed). Leave **Instruction** empty.

1. Open the character's **Tryll Voice Profile** data asset.

1. Set the TTS model to the cloning model and **Reference Voice** to the neutral clip.

1. Under **Delivery Modes**, add one element per mood with its **Name** and **Reference Voice**. Leave **Instruction** empty.

```
from tryll_client import DeliveryMode, VoiceProfile

guard = VoiceProfile(
    tts_model_name="Qwen3-TTS 12Hz 1.7B Base (Q4_K_M)",
    reference_voice="voices/guard/neutral.wav",
    delivery_modes=[
        DeliveryMode(name="angry",  reference_voice="voices/guard/angry.wav"),
        DeliveryMode(name="afraid", reference_voice="voices/guard/afraid.wav"),
    ])
agent = client.create_agent(graph, voice=guard)
```

A mode can also set **Speed Scale**, for example `1.15` to make `afraid` a little faster.

By default, the server prepares every clip in the profile when the agent is created, so the first sentence in a new mood has no extra delay. Set **Prepare** to `Lazy` to prepare each clip on its first use instead. A bad or missing clip path, in the profile or in any mode, fails agent creation with `3023`, and the message names the field.

## 4. Switch the mood at runtime

```
await agentComp.SetDeliveryModeAsync("angry");    // the guard loses his temper
await agentComp.SetDeliveryModeAsync("neutral");  // and calms down again
```

Call **Set Delivery Mode** on the **Tryll Agent** component with the mode name. It reports completion through its delegate.

```
agent.set_delivery_mode("angry")
agent.set_delivery_mode("neutral")
```

```
agent.SetDeliveryMode("angry", 1.0f);
```

- **It applies from the next sentence.** The call is accepted at any time, even mid-turn. A sentence already sent to the TTS model keeps its mood.
- **`neutral` is built in.** It always exists and means the profile's own **Reference Voice**.
- **Intensity does not blend clips.** A clip is all or nothing. On Pocket TTS, `intensity` only scales the mode's **Speed Scale**; on Qwen3-TTS Base it does nothing.
- **An unknown name** fails with `3024 UnknownDeliveryMode`, and the current mode stays.

To try a mood without code, use the agent component's **Voice Delivery** override in the Inspector (Unity) or Details panel (Unreal). Its dropdown lists `neutral` and the profile's modes, and changing it on a live agent switches the mood at once.

### Decide when to switch

Tryll does not detect the mood for you: only the API switches the mode, and the language model cannot. Your game owns the character's emotional state, the same way it owns health or quest progress, and calls `SetAgentDeliveryMode` when it changes. Pass the same mood into the prompt too, so that *what* the character says matches *how* it sounds. See [Talking NPC → An NPC who shows emotion](https://docs.tryllengine.com/v0.9/use-cases/talking-npc.html#step-4-an-npc-who-shows-emotion).

## Example: the Unity demo

The Unity demo's **EmotionalVoiceCharacter** is a complete example. Kira, a K-pop singer giving a magazine interview, speaks with one Qwen3-TTS voice and eight moods:

- **Voice profile:** `Assets/Voices/VoiceCharacterWithEmotions`, on `Qwen3-TTS 12Hz 1.7B Base (Q4_K_M)`.
- **Neutral clip:** `voice/young_female/female_young_calm.wav`.
- **Delivery modes:** `angry`, `very_angry`, `fear`, `hysteria`, `joy`, `sad`, `flirty` and `terror`, each with its own clip in `TryllStorage/voice/young_female/`.
- **Workflow:** `Assets/Workflows/EmotionalVoiceCharacter`.

The clips were made with Qwen3-TTS VoiceDesign, with one description per mood. To hear it, open `DemoScene`, select the **EmotionalVoiceCharacter** object, pick a mood in the agent component's **Voice Delivery** dropdown, then enter Play mode and talk to Kira. Change the mood while she speaks to hear the switch at the next sentence.

## Other TTS models

How a mode expresses a mood depends on the model:

| Model                      | What a mood is                                                       |
| -------------------------- | -------------------------------------------------------------------- |
| Qwen3-TTS Base, Pocket TTS | A reference clip per mood (this page)                                |
| Maya1                      | A text **Instruction**, such as `angry` or `whisper`, with intensity |
| Supertonic 3, Orpheus      | No emotion control: a mode can only change speed, or stay empty      |

The details for each model are in [Voice Profile → Delivery modes](https://docs.tryllengine.com/v0.9/reference/voice-profile.html#delivery-modes).

## Related

- [Clone a Voice from an Audio Sample](https://docs.tryllengine.com/v0.9/how-to/clone-a-voice.html)
- [Design a Voice Clip with Qwen3-TTS](https://docs.tryllengine.com/v0.9/how-to/design-a-voice-with-qwen3-tts.html)
- [Reference: Voice Profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html) — every field, delivery modes, intensity, clip preparation
- [Reference: TTS Models](https://docs.tryllengine.com/v0.9/reference/tts-models.html) — delivery modes by family
- [Use case: Talking NPC](https://docs.tryllengine.com/v0.9/use-cases/talking-npc.html) — where emotion fits in a full character
