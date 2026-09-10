# TryllSpeaker

**Type:** `MonoBehaviour`\
**Namespace:** `Tryll.Client`\
**Source:** `Runtime/TryllSpeaker.cs`

Plays streaming TTS audio from a [`TryllAgent`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/index.html#tryllagent) through an `AudioSource`. Wire it up by assigning this component to the `Speaker` slot on a [`TryllAgentComponent`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-agent-component.html), let `TryllAgentComponent` auto-provision it when the graph contains a `GenerateAndSpeak` / `Speak` node, or attach it to a code-created agent:

```
var speaker = GetComponent<TryllSpeaker>(); // or AddComponent
agent.SetOnTtsAudioFormat(speaker.HandleFormat);
agent.SetOnTtsAudio(speaker.HandleAudio);
```

______________________________________________________________________

## Requirements

`[RequireComponent(typeof(AudioSource))]` — Unity provisions an `AudioSource` automatically when `TryllSpeaker` is added to a GameObject.

______________________________________________________________________

## Inspector fields

| Field  | Type          | Default | Description                                                                                                                                            |
| ------ | ------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `_src` | `AudioSource` | auto    | The `AudioSource` used for playback. Populated automatically in `Awake`; override only when using a pre-existing `AudioSource` on the same GameObject. |

______________________________________________________________________

## Runtime state

| Member            | Type   | Description                                                                                                                                                                                                                                                                                                                                                                                              |
| ----------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `HasPendingAudio` | `bool` | True while decoded audio is still waiting in the chunk queue (or the front chunk is only partially played). After the turn's `TurnComplete` — when no further audio arrives — a `false` value means playback has drained down to the small (~100 ms) streaming clip buffer. Useful for hands-free flows that must not re-open the microphone while the character is still audibly speaking. Main thread. |
| `StopPlayback()`  | `void` | Cancels the current turn's playback: drops all queued audio and ignores further audio frames until the next turn's format announcement. Safe to call repeatedly and when nothing is playing. Main thread.                                                                                                                                                                                                |

`AudioSource.isPlaying` is not a playback signal here: the streaming clip loops and keeps "playing" silence between turns.

______________________________________________________________________

## Lifecycle and threading

| Method                                                        | Thread       | Description                                                                                                                                                                                                                                                                                                                                                                        |
| ------------------------------------------------------------- | ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `HandleFormat(nodeName, sampleRate, channels, bitsPerSample)` | Main thread  | Public. Called when the server announces the audio format (via `TryllAgent.SetOnTtsAudioFormat` or `TryllAgentComponent`). Creates or reuses a streaming `AudioClip`. `nodeName` is accepted for signature compatibility and unused — this speaker mixes all producers into one `AudioSource`. Only mono int16 PCM is supported; other formats are logged as warnings and ignored. |
| `HandleAudio(nodeName, ArraySegment<short>)`                  | Main thread  | Public. Converts int16 → float32, enqueues a chunk. `nodeName` is unused (same as `HandleFormat`). Drops the newest chunk with a logged warning when the queue is full (`kMaxQueuedChunks = 1024`).                                                                                                                                                                                |
| `PcmReader(float[])`                                          | Audio thread | Drains the chunk queue into Unity's audio buffer. Zero-fills the tail on underrun.                                                                                                                                                                                                                                                                                                 |

`HandleFormat` and `HandleAudio` fire on the Unity main thread (drained by `TryllClient.Update`). `PcmReader` runs on the Unity audio thread. The two share a chunk queue protected by a brief lock.

______________________________________________________________________

## End-of-stream

`TtsAudioFrame` carries no `is_final` flag — the turn ends with `TurnComplete`. The queue drains naturally; there is no explicit flush. When the `AudioSource` reaches the tail of the queue it will play silence until the next turn starts a new stream.

______________________________________________________________________

## Overflow behaviour

TTS synthesis runs well above realtime for typical responses. When many chunks arrive faster than they are played the queue fills. On overflow the **newest** incoming chunk is dropped (not the oldest), producing a minor glitch rather than a truncated sentence. A single warning is logged per turn when overflow occurs.

______________________________________________________________________

## Lifetime and ownership

- `TryllSpeaker` does not own its `TryllAgent`. `TryllAgentComponent` owns both when you use the component path; a code-created agent owns the wiring via `SetOnTtsAudioFormat` / `SetOnTtsAudio`.
- `OnDisable` stops playback and clears the chunk queue.
- `OnDestroy` destroys the `AudioClip`.
- Safe to disable between turns (playback stops immediately, queue clears). Re-enabling before the next `HandleFormat` call is fine.

______________________________________________________________________

## See also

- [`TryllAgentComponent`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-agent-component.html) — manages the agent and routes TTS callbacks to this component
- [How-to: Add Voice Output to an Agent](https://docs.tryllengine.com/v0.8/how-to/add-voice-to-an-agent.html)
- [Concept: TTS and Voice Output](https://docs.tryllengine.com/v0.8/concepts/tts-and-voice-output.html)
- [Reference: GenerateAndSpeak node](https://docs.tryllengine.com/v0.8/reference/nodes/generate-and-speak.html)
