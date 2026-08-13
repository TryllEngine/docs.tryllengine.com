# TryllSpeaker

**Type:** `MonoBehaviour`\
**Namespace:** `Tryll.Client`\
**Source:** `Runtime/TryllSpeaker.cs`

Plays streaming TTS audio from a [`TryllAgent`](https://docs.tryllengine.com/reference/client-api/unity/index.html#tryllagent) through an `AudioSource`. Wire it up by assigning this component to the `Speaker` slot on a [`TryllAgentComponent`](https://docs.tryllengine.com/reference/client-api/unity/tryll-agent-component.html), or let `TryllAgentComponent` auto-provision it when the graph contains a `GenerateAndSpeak` node.

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

There are no public runtime properties. Check `AudioSource.isPlaying` on the same GameObject to poll playback state.

______________________________________________________________________

## Lifecycle and threading

| Method                                              | Thread       | Description                                                                                                                                                                                                |
| --------------------------------------------------- | ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `HandleFormat(sampleRate, channels, bitsPerSample)` | Main thread  | Called by `TryllAgentComponent` when the server announces the audio format. Creates or reuses a streaming `AudioClip`. Only mono int16 PCM is supported; other formats are logged as warnings and ignored. |
| `HandleAudio(ArraySegment<short>)`                  | Main thread  | Converts int16 → float32, enqueues a chunk. Drops the newest chunk with a logged warning when the queue is full (`kMaxQueuedChunks = 1024`).                                                               |
| `PcmReader(float[])`                                | Audio thread | Drains the chunk queue into Unity's audio buffer. Zero-fills the tail on underrun.                                                                                                                         |

`HandleFormat` and `HandleAudio` fire on the Unity main thread (drained by `TryllClient.Update`). `PcmReader` runs on the Unity audio thread. The two share a chunk queue protected by a brief lock.

______________________________________________________________________

## End-of-stream

`TtsAudioFrame` carries no `is_final` flag — the turn ends with `TurnComplete`. The queue drains naturally; there is no explicit flush. When the `AudioSource` reaches the tail of the queue it will play silence until the next turn starts a new stream.

______________________________________________________________________

## Overflow behaviour

TTS synthesis runs well above realtime for typical responses. When many chunks arrive faster than they are played the queue fills. On overflow the **newest** incoming chunk is dropped (not the oldest), producing a minor glitch rather than a truncated sentence. A single warning is logged per turn when overflow occurs.

______________________________________________________________________

## Lifetime and ownership

- `TryllSpeaker` does not own its `TryllAgent`; `TryllAgentComponent` owns both.
- `OnDisable` stops playback and clears the chunk queue.
- `OnDestroy` destroys the `AudioClip`.
- Safe to disable between turns (playback stops immediately, queue clears). Re-enabling before the next `HandleFormat` call is fine.

______________________________________________________________________

## See also

- [`TryllAgentComponent`](https://docs.tryllengine.com/reference/client-api/unity/tryll-agent-component.html) — manages the agent and routes TTS callbacks to this component
- [How-to: Add Voice Output to an Agent](https://docs.tryllengine.com/how-to/add-voice-to-an-agent.html)
- [Concept: TTS and Voice Output](https://docs.tryllengine.com/concepts/tts-and-voice-output.html)
- [Reference: GenerateAndSpeak node](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html)
