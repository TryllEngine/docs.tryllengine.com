# TryllVoiceProfile

**Type:** `ScriptableObject` (sealed)\
**Namespace:** `Tryll.Client`\
**Source:** `Runtime/TryllVoiceProfile.cs`, `Runtime/TryllVoiceProfileData.cs`\
**Menu:** Create → Tryll → Voice Profile

A character's voice as an asset: which TTS model speaks, the base identity (speaker, reference clip or description), speed and language, whether a viseme track is emitted, and named **delivery modes** (moods such as `angry` or `calm`) that game code switches at runtime.

Assign the profile to a [`TryllAgentComponent`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-agent-component.html)'s **Voice** field. Every speaking node of that agent (`Speak`, `GenerateAndSpeak`) uses it; the nodes hold no voice settings. Because the voice lives on the agent and not in the workflow, one workflow asset can drive many characters, each with its own profile.

A workflow with a speaking node cannot create an agent without a voice: the server rejects it with `AgentVoiceRequired` (3025). There is no default voice and no default model. A workflow that does not speak ignores the profile and loads no TTS model.

______________________________________________________________________

## Fields

The asset holds one `TryllVoiceProfileData` in its `Data` field.

| Field                 | Type                      | Default | Description                                                                                                                                                                                              |
| --------------------- | ------------------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `TtsModelName`        | `string`                  | —       | **Required.** TTS model catalog name, picked from the registered TTS models.                                                                                                                             |
| `SpeakerId`           | `int`                     | `0`     | Speaker / style index for multi-speaker models (Supertonic 3: 0–9; Orpheus, Maya1: preset index). Must be below the model's speaker count.                                                               |
| `ReferenceVoice`      | `string`                  | empty   | Reference clip (`.wav`) for clone models (Pocket TTS, Chatterbox, Qwen3-TTS Base), relative to the storage root. Locale-aware: `voices/loc/<locale>/` is tried first. Empty = the model's default voice. |
| `VoiceDescription`    | `string`                  | empty   | Maya1: natural-language identity of the voice. Overrides `SpeakerId`.                                                                                                                                    |
| `Speed`               | `float`                   | `1.0`   | Speech-rate multiplier, 0.1–3.0. Must stay 1.0 on models without speed control.                                                                                                                          |
| `Lang`                | `string`                  | empty   | Language for multilingual models that pick it per call. Empty = session locale, then the model default. Rejected on models with a fixed language.                                                        |
| `Expressiveness`      | `float`                   | `0.5`   | Chatterbox exaggeration, 0.0–2.0, used by `neutral` and by modes that do not set their own. Ignored by other models.                                                                                     |
| `Prepare`             | `TryllVoicePreparation`   | `Eager` | When reference clips are prepared: `Eager` (all clips when the voice is set, so the first sentence in a new mode has no extra latency) or `Lazy` (on first use).                                         |
| `EmitVisemes`         | `bool`                    | `false` | Emit a viseme track for every sentence the agent speaks. Turn on for characters with [`TryllLipSync`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-lip-sync.html).                 |
| `DeliveryModes`       | `List<TryllDeliveryMode>` | empty   | Named deliveries (below).                                                                                                                                                                                |
| `DefaultDeliveryMode` | `string`                  | empty   | Mode active after the voice is set. Empty = `neutral`.                                                                                                                                                   |

### TryllDeliveryMode

| Field            | Type     | Default | Description                                                                                                                                                                                                |
| ---------------- | -------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Name`           | `string` | —       | Mode name: 1–32 characters of `a-z`, `0-9`, `_`, unique in the profile. `neutral` is built in and cannot be defined.                                                                                       |
| `ReferenceVoice` | `string` | empty   | Clone models: a clip of this character in this mood. Replaces the profile clip while the mode is active.                                                                                                   |
| `Instruction`    | `string` | empty   | Maya1: sustained delivery clause. A known keyword (`angry`, `sad`, `whisper`, …) becomes its tuned clause. A ticked Text ID in this field is not resolved; the text resolver only walks graph node params. |
| `Expressiveness` | `float`  | `-1`    | Chatterbox exaggeration for this mode, 0.0–2.0. Negative = the profile value.                                                                                                                              |
| `SpeedScale`     | `float`  | `1.0`   | Multiplies the profile speed while the mode is active, 0.5–2.0. Must stay 1.0 on models without speed control.                                                                                             |

A mode with no fields set behaves like `neutral`. This lets every profile define the same mode names, so game code that asks for `angry` works on any voice, including models with no emotion channel.

A field the model cannot use (a reference clip on Supertonic, an instruction on Qwen3-TTS) is an error: the server rejects the profile with `InvalidVoiceProfile` (3023) when the voice is set.

______________________________________________________________________

## Inspector

A cloning-model profile with one reference clip per emotion. Qwen3-TTS has no speaker presets, so no **Speaker** field is shown.

The custom Inspector shows only what the chosen model can use, read from the catalog's TTS capabilities:

- **Speaker** is a preset dropdown for models with named presets (Orpheus, Maya1), a slider for multi-speaker models (Supertonic 3), and hidden for single-speaker models.
- **Reference Voice**, **Voice Description**, **Speed**, **Lang** and **Expressiveness** appear only for models that support them. A field that holds a value the model cannot use stays visible so you can clear it.
- **Delivery Modes** is a reorderable list whose rows apply the same rules.
- **Default Delivery Mode** is a dropdown of the defined modes.
- Validation findings (missing model, bad mode names, unsupported fields, missing clip files) are shown at the bottom.

When the catalog cannot be read, every field is shown.

**Assets ▸ Tryll ▸ Validate Workflow** also validates selected voice profiles. A Windows Standalone build fails when a profile names a model that is not marked Production in the Model Manager.

______________________________________________________________________

## Using a profile from code

```
// Spawning a character: set the voice before the agent is created.
var agent = go.AddComponent<TryllAgentComponent>();
agent.AutoCreateOnConnect = false;
agent.WorkflowAsset = merchantWorkflow;
agent.Voice = merchantVoice;          // TryllVoiceProfile asset
agent.CreateAgent();

// Mood: any time, including mid-turn; applies from the next sentence.
var err = await agent.SetDeliveryModeAsync("angry", intensity: 0.8f);

// Identity: idle (or paused) only.
err = await agent.SetVoiceAsync(disguisedVoice);
```

Without a component, build a `TryllVoiceProfileData` and pass it to [`TryllClient.RequestCreateAgentAsync`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-client.html) (`voice:`), then use `TryllAgent.SetVoiceAsync` / `SetDeliveryModeAsync`.

```
public sealed class TryllVoiceProfileData
{
    public const string NeutralMode = "neutral";
    public TryllVoiceProfileData Clone();
    public Tryll.VoiceProfileT ToWire();
    public static TryllVoiceProfileData FromWire(Tryll.VoiceProfileT wire);
    public static bool IsValidModeName(string name);
}
```

| Error                        | When                                                                                                       |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `InvalidVoiceProfile` (3023) | A field fails validation or the model cannot use it. The message names the field and, for modes, the mode. |
| `UnknownDeliveryMode` (3024) | `SetDeliveryModeAsync` names a mode the profile does not define.                                           |
| `AgentVoiceRequired` (3025)  | A speaking workflow was created without a voice, or `SetDeliveryModeAsync` on an agent with no voice.      |
| `AgentBusy` (3004)           | `SetVoiceAsync` while a turn is running.                                                                   |

______________________________________________________________________

## See also

- [`TryllAgentComponent`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-agent-component.html)
- [`TryllSpeaker`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-speaker.html)
- [`TryllLipSync`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-lip-sync.html)
- [Concept: TTS and Voice Output](https://docs.tryllengine.com/v0.9/concepts/tts-and-voice-output.html)
- [Reference: Speak node](https://docs.tryllengine.com/v0.9/reference/nodes/speak.html)
- [Reference: GenerateAndSpeak node](https://docs.tryllengine.com/v0.9/reference/nodes/generate-and-speak.html)
