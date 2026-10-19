# TryllLipSync

**Type:** `MonoBehaviour` (sealed)\
**Namespace:** `Tryll.Client`\
**Source:** `Runtime/TryllLipSync.cs`\
**Menu:** Add Component → Tryll → Tryll Lip Sync

Plays an agent's viseme track on a face with ARKit blendshapes, in step with the audio played by a [`TryllSpeaker`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-speaker.html). The agent's [voice profile](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-voice-profile.html) must have **Emit Visemes** on — see [Add Lip Sync to a Character](https://docs.tryllengine.com/v0.9/how-to/add-lip-sync-to-a-character.html).

______________________________________________________________________

## Requirements

- `[RequireComponent(typeof(TryllSpeaker))]` — the speaker on the same GameObject is the playback clock. Assign it to the [`TryllAgentComponent`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-agent-component.html)'s **Speaker** field so the agent's audio plays through it.
- A [`TryllAgentComponent`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-agent-component.html) — the source of the viseme track. A warning is logged in `Awake` if none is assigned or found on the GameObject.
- A [`TryllVoiceProfile`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-voice-profile.html) on that component with **Emit Visemes** on. Without it no viseme track arrives: the Inspector shows a warning, and one is logged when the agent is bound.
- Face meshes whose blendshapes use ARKit names.

______________________________________________________________________

## Inspector fields

### Source

| Field      | Type                  | Default         | Description                                                                              |
| ---------- | --------------------- | --------------- | ---------------------------------------------------------------------------------------- |
| `_agent`   | `TryllAgentComponent` | same GameObject | Agent whose viseme track is played. Re-binds automatically when the agent is re-created. |
| `_speaker` | `TryllSpeaker`        | same GameObject | Speaker whose playback position the face follows.                                        |

### Face

| Field        | Type                        | Default | Description                                                                                                       |
| ------------ | --------------------------- | ------- | ----------------------------------------------------------------------------------------------------------------- |
| `_renderers` | `List<SkinnedMeshRenderer>` | empty   | Meshes to drive. Bound at `Start` when set.                                                                       |
| `_faceRoot`  | `Transform`                 | none    | Parent of the face meshes. When **Renderers** is empty, every `SkinnedMeshRenderer` under it is bound at `Start`. |

Set one of the two, or call `BindFace` at runtime; with neither, no mesh is driven.

Only blendshapes named by the **Map** are bound and written; every other blendshape on those meshes is left untouched for other animation to drive. `MappedShapes` counts the bound ones.

### Clock

| Field                      | Type    | Default | Description                                                                                                                     |
| -------------------------- | ------- | ------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `_smoothPlaybackClock`     | `bool`  | `true`  | Smooths the speaker's playback position into a steady clock.                                                                    |
| `_clockWindowSeconds`      | `float` | `1.5`   | Window of the clock smoother.                                                                                                   |
| `_outputLeadTrimMs`        | `float` | `342`   | How far the streaming clip's buffer runs ahead of what is audible. Device-dependent; adjust when a large sync offset is needed. |
| `_compensateOutputLatency` | `bool`  | `true`  | Subtracts Unity's DSP buffer latency.                                                                                           |
| `_syncOffsetMs`            | `float` | `0`     | Manual sync correction. Positive moves the face earlier.                                                                        |

### Envelope

| Field                        | Type            | Default | Description                                                                                                                            |
| ---------------------------- | --------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `_attackDecayMs`             | `float`         | `50`    | Cross-fade between consecutive shapes.                                                                                                 |
| `_intensity`                 | `float` (0–1.5) | `0.4`   | Scale on every pose. `0` = no movement, `1` = the full pose table.                                                                     |
| `_smoothingPerSecond`        | `float` (0–120) | `16`    | Rate at which the face moves toward each shape. `0` disables smoothing.                                                                |
| `_closureSmoothingPerSecond` | `float` (0–200) | `60`    | Rate used while a lip closure (`PP`, `FF`) is active.                                                                                  |
| `_maxHoldMs`                 | `float`         | `400`   | Returns to rest when the last shape of a turn has been held this long (cancelled turns). `0` disables.                                 |
| `_compensateSmoothingLag`    | `bool`          | `true`  | Samples the track ahead by the delay the smoothing adds (`1000 / smoothing + attack / 2` ms), so tuning smoothing does not shift sync. |

### Mapping

| Field  | Type                    | Default        | Description                                                                                             |
| ------ | ----------------------- | -------------- | ------------------------------------------------------------------------------------------------------- |
| `_map` | `List<TryllVisemePose>` | built-in table | Pose per viseme, as ARKit channel weights. Left empty, it is filled with the built-in table at `Awake`. |

### Diagnostics and apply

| Field              | Type   | Default | Description                                                    |
| ------------------ | ------ | ------- | -------------------------------------------------------------- |
| `_logFrameArrival` | `bool` | `false` | Logs how far ahead of playback each viseme frame arrived.      |
| `_applyToMesh`     | `bool` | `true`  | When off, the pose is evaluated but not written to the meshes. |

______________________________________________________________________

## Public members

| Member                                    | Type                           | Description                                                                                                                                                                                       |
| ----------------------------------------- | ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Intensity`                               | `float` (get/set)              | Same as the **Intensity** field.                                                                                                                                                                  |
| `SmoothingPerSecond`                      | `float` (get/set)              | Same as the **Smoothing Per Second** field.                                                                                                                                                       |
| `AttackDecayMs`                           | `float` (get/set)              | Same as the **Attack Decay Ms** field.                                                                                                                                                            |
| `ApplyToMesh`                             | `bool` (get/set)               | Same as the **Apply To Mesh** field.                                                                                                                                                              |
| `BindFace(Transform faceRoot)`            | `void`                         | Re-binds to the meshes under `faceRoot` — call it after spawning or swapping the face at runtime.                                                                                                 |
| `FaceBound`                               | `bool`                         | True once at least one blendshape is bound.                                                                                                                                                       |
| `MappedShapes`                            | `int`                          | Number of bound blendshapes.                                                                                                                                                                      |
| `ActiveViseme`                            | `WireViseme`                   | Viseme being shown this frame.                                                                                                                                                                    |
| `AppliedJawOpen`                          | `float`                        | `jawOpen` value written this frame (0–1), after **Intensity** and smoothing; `0` if the mesh has no `jawOpen`. With **Apply To Mesh** off, the raw pose value before **Intensity** and smoothing. |
| `SmoothingLagMs`                          | `float`                        | Look-ahead currently applied for the smoothing delay.                                                                                                                                             |
| `OutputLatencyMs`                         | `double`                       | DSP buffer latency being subtracted.                                                                                                                                                              |
| `StatusLine`                              | `string`                       | One-line state for debug UI.                                                                                                                                                                      |
| `BuildDefaultOculusArkitMap()`            | `static List<TryllVisemePose>` | The built-in pose table.                                                                                                                                                                          |
| `VisemeCount`                             | `const int`                    | `15` — the number of visemes in the table.                                                                                                                                                        |
| `VisemeSetupProblem(TryllAgentComponent)` | `static string`                | Why the agent will send no viseme track (no voice profile, or Emit Visemes off), or `null` when it is set up.                                                                                     |

______________________________________________________________________

## Built-in pose table

The 15 visemes and the ARKit channels each one drives (before **Intensity** and the event weight are applied):

| Viseme | Sounds (as in)                                  | Channels                                                                      |
| ------ | ----------------------------------------------- | ----------------------------------------------------------------------------- |
| `sil`  | rest, pauses                                    | —                                                                             |
| `PP`   | **p**en, **b**at, **m**e                        | `mouthClose` .55, `mouthPressLeft`/`Right` .45, `mouthPucker` .15             |
| `FF`   | **f**an, **v**an                                | `jawOpen` .12, `mouthLowerDownLeft`/`Right` .55, `mouthFunnel` .15            |
| `TH`   | **th**in, **th**at                              | `jawOpen` .18, `mouthStretchLeft`/`Right` .25, `mouthUpperUpLeft`/`Right` .20 |
| `DD`   | **t**op, **d**og                                | `jawOpen` .22, `mouthClose` .15                                               |
| `kk`   | **k**ey, **g**o                                 | `jawOpen` .32                                                                 |
| `CH`   | **ch**in, **j**am, **sh**e, mea**s**ure         | `jawOpen` .18, `mouthFunnel` .45, `mouthPucker` .20                           |
| `SS`   | **s**un, **z**oo                                | `jawOpen` .08, `mouthStretchLeft`/`Right` .55                                 |
| `nn`   | **n**o, **l**ow, si**ng**                       | `jawOpen` .14, `mouthClose` .20                                               |
| `RR`   | **r**ed                                         | `jawOpen` .16, `mouthPucker` .35, `mouthFunnel` .15                           |
| `aa`   | f**a**ther, c**a**t, c**u**p, **a**bout, m**y** | `jawOpen` .72                                                                 |
| `E`    | b**e**d, s**ay**, h**er**                       | `jawOpen` .38, `mouthStretchLeft`/`Right` .45                                 |
| `ih`   | s**i**t, s**ee**, **y**es                       | `jawOpen` .28, `mouthStretchLeft`/`Right` .28                                 |
| `oh`   | l**aw**, g**o**, b**oy**                        | `jawOpen` .48, `mouthFunnel` .55, `mouthPucker` .25                           |
| `ou`   | y**ou**, b**oo**k, h**ow**, **w**e              | `jawOpen` .28, `mouthPucker` .72, `mouthFunnel` .35                           |

______________________________________________________________________

## Related types

| Type                       | Description                                                                                                                                                                                                  |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `TryllVisemeEvent`         | `readonly struct`: `SampleOffset` (`uint`, samples from the start of the turn's audio — its format announcement — across every speaking node of the turn), `Viseme` (`WireViseme`), `Weight` (`float`, 0–1). |
| `TryllAgent.OnTtsVisemes`  | `event Action<string, TryllVisemeEvent[]>` — fired on the main thread for each viseme frame (one per synthesized sentence). The array belongs to the invocation; copy it to keep it.                         |
| `WireViseme`               | `enum : byte` in namespace **`Tryll`** (add `using Tryll;`): `sil, PP, FF, TH, DD, kk, CH, SS, nn, RR, aa, E, ih, oh, ou`.                                                                                   |
| `TryllVisemePose`          | Serializable: `Viseme` + `Channels` (`TryllVisemeChannelWeight[]`).                                                                                                                                          |
| `TryllVisemeChannelWeight` | Serializable: `Channel` (ARKit blendshape name) + `Weight`.                                                                                                                                                  |

______________________________________________________________________

## Lifecycle and threading

- `Awake` fills the default map and builds the pose table; `Start` binds the meshes; `OnEnable` subscribes to the agent's viseme events.
- Each `LateUpdate` evaluates the pose at the current playback position and writes it to the meshes. Everything runs on the main thread.
- Each audio format announcement (one per agent turn) clears the previous track.
- `OnDisable` unsubscribes and resets the clock and smoothing.

______________________________________________________________________

## See also

- [How-to: Add Lip Sync to a Character](https://docs.tryllengine.com/v0.9/how-to/add-lip-sync-to-a-character.html)
- [`TryllSpeaker`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-speaker.html)
- [`TryllVoiceProfile`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-voice-profile.html)
- [Concept: TTS and Voice Output](https://docs.tryllengine.com/v0.9/concepts/tts-and-voice-output.html#lip-sync-visemes)
- [Reference: GenerateAndSpeak node](https://docs.tryllengine.com/v0.9/reference/nodes/generate-and-speak.html)
