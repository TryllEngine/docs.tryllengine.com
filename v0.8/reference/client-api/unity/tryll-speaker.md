# TryllSpeaker

**Type:** `MonoBehaviour` (sealed)\
**Menu:** Add Component → Tryll → Tryll Speaker\
**Namespace:** `Tryll.Client`\
**Source:** `Runtime/TryllSpeaker.cs`

Plays streaming TTS audio from a [`TryllAgent`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/index.html#tryllagent) through an `AudioSource`. Wire it up by assigning this component to the `Speaker` slot on a [`TryllAgentComponent`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-agent-component.html), let `TryllAgentComponent` find or add it when the graph contains a `GenerateAndSpeak` / `Speak` node (it uses a `TryllSpeaker` already on the GameObject, else adds one, at `CreateAgent`), or attach it to a code-created agent:

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
| `ConsumedSamples` | `long` | Playback clock: samples handed to the audio device since the stream started, at the current sample rate. Underrun silence is not counted; audio dropped unplayed (cancel, format change, disable) counts as played, so the clock never falls behind the stream. Restarts at 0 when the sample rate changes. Any thread.                                                                                  |
| `QueuedSamples`   | `long` | Samples still waiting in the queue, including the unplayed tail of the front chunk. `ConsumedSamples + QueuedSamples` at a format announcement is where that node's audio starts. Main thread.                                                                                                                                                                                                           |
| `StopPlayback()`  | `void` | Cancels the current playback: drops all queued audio and ignores further audio frames until the next speaking node's format announcement — usually the next turn, but a second speaking node in the same turn plays. Does not cancel the turn; call `Cancel` on the agent for that. Safe to call repeatedly and when nothing is playing. Main thread.                                                    |

`AudioSource.isPlaying` is not a playback signal here: the streaming clip loops and keeps "playing" silence between turns.

______________________________________________________________________

## Events

Both fire on the Unity main thread and are additive multicast (`+=`). `TryllLipSync` uses them; subscribe yourself to drive your own track.

| Event         | Signature                                                                  | When                                                                                                                             |
| ------------- | -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `OnTtsFormat` | `Action<string nodeName, int sampleRate, int channels, int bitsPerSample>` | Each format announcement (one per speaking node), after the mono-int16 check and after a sample-rate change has reset the clock. |
| `OnTtsPcm`    | `Action<string nodeName, ArraySegment<short> pcm>`                         | Each PCM chunk, before the format and `StopPlayback` checks. The segment is valid only during the callback.                      |

______________________________________________________________________

## Lifecycle and threading

| Method                                                        | Thread       | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ------------------------------------------------------------- | ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `HandleFormat(nodeName, sampleRate, channels, bitsPerSample)` | Main thread  | Public. Called when the server announces the audio format (via `TryllAgent.SetOnTtsAudioFormat` or `TryllAgentComponent`). Creates the streaming `AudioClip` on the first call and reuses it while the sample rate stays the same; a new rate rebuilds it, discards queued audio and restarts `ConsumedSamples` at 0. Lifts `StopPlayback` and raises `OnTtsFormat`. Outside Play mode (for example the Tryll Chat window) it logs an error and plays nothing. `nodeName` is accepted for signature compatibility and unused — this speaker mixes all producers into one `AudioSource`. Only mono int16 PCM is supported; other formats are logged as warnings and ignored. |
| `HandleAudio(nodeName, ArraySegment<short>)`                  | Main thread  | Public. Raises `OnTtsPcm`, then converts int16 → float32 and enqueues the chunk. Ignored before the first format, and after `StopPlayback` until the next format. `nodeName` is unused (same as `HandleFormat`). Drops the newest chunk with a logged warning when the queue is full (`kMaxQueuedChunks = 1024`).                                                                                                                                                                                                                                                                                                                                                           |
| `PcmReader(float[])`                                          | Audio thread | Drains the chunk queue into Unity's audio buffer. Zero-fills the tail on underrun.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |

`HandleFormat` and `HandleAudio` fire on the Unity main thread (drained by `TryllClient.Update`). `PcmReader` runs on the Unity audio thread. The two share a chunk queue protected by a brief lock.

______________________________________________________________________

## End-of-stream

`TtsAudioFrame` carries no `is_final` flag — the turn ends with `TurnComplete`. The queue drains naturally; there is no explicit flush. When the `AudioSource` reaches the tail of the queue it will play silence until the next turn starts a new stream.

______________________________________________________________________

## Overflow behaviour

TTS synthesis runs well above realtime for typical responses. When many chunks arrive faster than they are played the queue fills. On overflow the **newest** incoming chunk is dropped (not the oldest), producing a minor glitch rather than a truncated sentence. A single warning is logged per speaking node's stream (the counter resets at each format announcement).

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
