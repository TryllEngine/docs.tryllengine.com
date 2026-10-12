# Add Lip Sync to a MetaHuman (Unreal)

Make a MetaHuman's mouth move in time with an agent's Tryll TTS voice. This uses the **Tryll MetaHuman Lip Sync** plugin (`UnrealPlugin/TryllMetaHumanLipSync`), a separate plugin installed alongside [TryllClient](https://docs.tryllengine.com/v0.8/reference/client-api/unreal/cpp/index.html). It has two solvers, and both feed the face through Live Link:

- **Neural** (default) runs Epic's realtime speech-to-animation model on the CPU over the TTS audio. It drives the whole face.
- **Viseme** plays the server's viseme track through Epic's ARKit mapping. It drives the mouth and jaw, and runs no model.

See [Choose a solver](#choose-a-solver).

## Requirements

- Unreal Engine 5.7, Windows 64-bit.
- The TryllClient plugin with voice output working — see [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.8/how-to/add-voice-to-an-agent.html).
- A MetaHuman in your project. Both kinds work: characters assembled in the in-engine MetaHuman Creator and characters downloaded from Quixel Bridge.

## Enable and set up

1. Copy the `TryllMetaHumanLipSync` folder from the Tryll release into your project's `Plugins/` folder, next to `TryllClient`, and enable **Tryll MetaHuman Lip Sync** in **Edit > Plugins**. It is off by default.
1. Open **Project Settings > Plugins > Tryll MetaHuman Lip Sync** and click **Set Up**. The status line changes to *Set up.*

Set Up copies Epic's speech-to-animation model from your engine installation into your project as `Content/TryllMetaHumanLipSync/solar-1-2700_blend-0v65.uasset` and configures it for CPU inference. Nothing is redistributed by Tryll. The asset is generated, so add it to `.gitignore` and run Set Up on each fresh clone.

Until Set Up has run, **Play** is blocked with a notification naming the step, and packaging a Win64 game target fails at build time with the same message. This applies with either solver. Dedicated server targets are not affected.

### Fresh clones and build machines

Run Set Up from the command line instead of the editor:

```
UnrealEditor-Cmd.exe <Project>.uproject -run=TryllMetaHumanLipSyncSetup
UnrealEditor-Cmd.exe <Project>.uproject -run=TryllMetaHumanLipSyncSetup -verify
```

Both exit with `0` when the project is set up and `1` otherwise. `-verify` checks without changing anything.

## Add lip sync to a character

Open the MetaHuman's Blueprint and add two components:

1. **Tryll Speaker**, configured with a TTS voice as in [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.8/how-to/add-voice-to-an-agent.html).
1. **Tryll MetaHuman Lip Sync**.

That's all. When the character starts, the component finds the speaker and the skeletal mesh named `Face` on the same actor, and the mouth follows the voice.

### Other layouts

If the components are not on the MetaHuman itself, set **Target MetaHuman** to the character. If the owning actor has several Tryll Speakers, set **Speaker** to the one to follow. With nothing set, the component also looks in the owner's child actors for the face.

## Choose a solver

Set **Solver** on the component. It is read once when the character starts.

|                            | Neural                                                 | Viseme                                                         |
| -------------------------- | ------------------------------------------------------ | -------------------------------------------------------------- |
| Drives                     | The whole face                                         | Mouth and jaw                                                  |
| Input                      | The TTS audio                                          | The server's viseme track                                      |
| Graph change               | None                                                   | Turn on `emit_visemes` on the speaking `GenerateAndSpeak` node |
| CPU per speaking character | About 4 to 5 ms per 20 ms of audio                     | No model runs                                                  |
| Face timing                | Trails the audio by the lookahead plus one 20 ms block | Follows the audio as it plays                                  |

### Use the Viseme solver

1. On the speaking `GenerateAndSpeak` node, tick **Emit Visemes**. Without it no track arrives and the mouth stays still, with no error.
1. Set **Solver** to **Viseme**.
1. Check **ARKit Mapping** points at Epic's `PA_MetaHuman_ARKit_Mapping` pose asset. The default is `/Game/MetaHumans/Common/Face/ARKit/PA_MetaHuman_ARKit_Mapping`, where exported MetaHumans keep it. The MetaHuman Character plugin also ships one in its `Face/ARKit` folder.

For what the viseme track is and how to tune it, see [Add Lip Sync to a Character](https://docs.tryllengine.com/v0.8/how-to/add-lip-sync-to-a-character.html). Its Unreal **Tryll Lip Sync** component uses the same viseme settings on faces with ARKit morph targets.

## Settings

| Setting          | Default | Notes                                                                                                                      |
| ---------------- | ------- | -------------------------------------------------------------------------------------------------------------------------- |
| Target MetaHuman | unset   | The character to animate. Unset means the owner, then its child actors.                                                    |
| Speaker          | unset   | The Tryll Speaker to follow. Unset means the only one on the owner.                                                        |
| Solver           | Neural  | **Neural** or **Viseme**. Read once when the character starts.                                                             |
| Subject Name     | empty   | Live Link subject name. Empty derives `TryllMetaHumanLipSync_<actor label>`, suffixed only if that name is already in use. |

Neural solver only:

| Setting        | Default | Notes                                                                                                   |
| -------------- | ------- | ------------------------------------------------------------------------------------------------------- |
| Mood           | Neutral | Emotional tone applied by the model.                                                                    |
| Mood Intensity | 1.0     | 0 to 1.                                                                                                 |
| Lookahead      | 80 ms   | 80 to 240. Higher is smoother and adds the same amount of latency. Read once when the character starts. |

Change mood while the game runs with **Set Mood** (`SetMood(Mood, Intensity)`).

Viseme solver only:

| Setting                                        | Default   | Notes                                                                                                                  |
| ---------------------------------------------- | --------- | ---------------------------------------------------------------------------------------------------------------------- |
| Viseme Settings > Intensity                    | 1.0       | Scales every mouth pose. `0` keeps the mouth at rest.                                                                  |
| Viseme Settings > Smoothing Per Second         | 16        | How quickly the mouth moves toward each new shape. Lower is softer, higher is snappier.                                |
| Viseme Settings > Closure Smoothing Per Second | 60        | Faster rate for lip closures (p, b, m, f, v), so the lips meet on short sounds.                                        |
| Viseme Settings > Attack Decay Ms              | 50        | Cross-fade between consecutive shapes.                                                                                 |
| Viseme Settings > Max Hold Ms                  | 400       | Returns the mouth to rest if a turn is cancelled mid-sentence.                                                         |
| Viseme Settings > Sync Offset Ms               | 0         | Shifts the mouth against the audio. Positive moves it earlier.                                                         |
| Viseme Settings > Compensate Output Latency    | on        | Holds the mouth back by the audio engine's output buffer, so it lines up with the sound as heard.                      |
| Viseme Settings > Compensate Smoothing Lag     | on        | Looks ahead by the delay the smoothing adds, so changing smoothing does not shift sync.                                |
| Viseme Poses                                   | empty     | The pose for each viseme as ARKit channel weights. Empty uses the built-in table. Read once when the character starts. |
| ARKit Mapping                                  | see above | Epic's ARKit to MetaHuman pose asset.                                                                                  |

Viseme Settings can be changed while the game runs.

## Troubleshooting

Every message is logged under `LogTryllMetaHumanLipSync` with the owning actor's name. **Get Lip Sync Status** returns the same state from Blueprint, and **On Lip Sync Status Changed** fires when it changes.

| Status                 | Cause                                                                              | Fix                                                      |
| ---------------------- | ---------------------------------------------------------------------------------- | -------------------------------------------------------- |
| `SpeakerNotFound`      | No Tryll Speaker on the owner.                                                     | Add one, or set **Speaker**.                             |
| `SpeakerAmbiguous`     | More than one Tryll Speaker on the owner.                                          | Set **Speaker**.                                         |
| `FaceNotFound`         | No skeletal mesh named `Face` on the owner, its child actors, or Target MetaHuman. | Set **Target MetaHuman** to the character.               |
| `LiveLinkUnavailable`  | The Live Link plugin is disabled.                                                  | Enable it.                                               |
| `AnimBlueprintMissing` | The plugin's face AnimBP is missing from its content.                              | Reinstall the plugin.                                    |
| `ModelMissing`         | Set Up has not run in this project.                                                | Run Set Up.                                              |
| `ModelLoadFailed`      | The model asset exists but ONNX Runtime rejected it.                               | See the NNE log lines just above the message.            |
| `InferenceFailed`      | ONNX Runtime failed while the character was speaking.                              | See the NNE log lines just above the message.            |
| `ArkitMappingMissing`  | Viseme solver: the **ARKit Mapping** asset could not be loaded.                    | Point **ARKit Mapping** at `PA_MetaHuman_ARKit_Mapping`. |

With the Viseme solver, a status of `Bound` and a mouth that never moves means no viseme track is arriving: turn on **Emit Visemes** on the speaking `GenerateAndSpeak` node. A `Speak` node sends no track.

## Performance

With the Neural solver, each speaking character costs about 4 to 5 ms of CPU per 20 ms of audio, measured on a Ryzen 9 7900X. Characters that are not speaking cost nothing. The face trails the audio by the lookahead plus one 20 ms block.

The Viseme solver runs no model.

## What it does to the character

The component sets the anim class on the `Face` mesh to the plugin's own face AnimBP, `ABP_TryllMetaHumanLipSyncFace`. It copies the neck and spine pose from the Body mesh, reads the Live Link subject, and provides a montage slot. With the Viseme solver, the AnimBP passes the Live Link ARKit curves through the **ARKit Mapping** pose asset, the same way Epic's `ABP_MH_LiveLink` handles ARKit input. Whatever face AnimBP the character had is not used while the component is on it. The MetaHuman's rig logic runs as normal in the mesh's post-process AnimBP, so no MetaHuman assets are referenced by the plugin, other than the ARKit Mapping you point it at.

## Related

- [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.8/how-to/add-voice-to-an-agent.html)
- [Concept: TTS and Voice Output](https://docs.tryllengine.com/v0.8/concepts/tts-and-voice-output.html)
