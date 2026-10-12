# Unity Client API

**Package:** `com.tryll.client`\
**Namespace:** `Tryll.Client`\
**Target:** Unity 6 (6000.x), .NET Standard 2.1, Windows Standalone + Editor

> The Unity client is a hand-authored C# port of the Unreal plugin. The public surface is documented on this page and the entity pages linked below. Internal types under `Tryll.Client.Internal` are not part of the public API.

______________________________________________________________________

## Inspector entities

These types appear in the Inspector and have dedicated reference pages.

| Type                                                                                                                      | Kind               | Description                                                                                                           |
| ------------------------------------------------------------------------------------------------------------------------- | ------------------ | --------------------------------------------------------------------------------------------------------------------- |
| [TryllClient](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-client.html)                             | `MonoBehaviour`    | Singleton session manager. Created automatically — do not add it to a scene.                                          |
| [TryllAgentComponent](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-agent-component.html)            | `MonoBehaviour`    | Drop-in component that owns and drives one agent on a GameObject.                                                     |
| [TryllSpeaker](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-speaker.html)                           | `MonoBehaviour`    | Plays streaming TTS audio from an agent through an `AudioSource`. Assign to a `TryllAgentComponent`'s `Speaker` slot. |
| [TryllLipSync](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-lip-sync.html)                          | `MonoBehaviour`    | Plays an agent's viseme track on ARKit face blendshapes, in step with a `TryllSpeaker`.                               |
| [TryllVoiceInputComponent](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-voice-input-component.html) | `MonoBehaviour`    | Microphone capture component that streams audio to the STT pipeline.                                                  |
| [TryllWorkflowAsset](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-workflow-asset.html)              | `ScriptableObject` | Project asset that stores a `TryllGraphDescription`.                                                                  |
| [TryllRuntimeSettings](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-runtime-settings.html)          | `ScriptableObject` | Package settings (server path, host, port). Edit via **Project Settings → Tryll Client**.                             |

______________________________________________________________________

## TryllAgent

`IDisposable` handle for a server-side agent. Obtained from `TryllClient.RequestCreateAgentAsync` or from `TryllAgentComponent.Agent`.

```
public ulong AgentId { get; }
public bool  IsValid { get; }

// Actions
public void SendMessage(string text);
public void Cancel(TryllCancelMode mode = TryllCancelMode.StopAndKeep);
public void ChangeParams(string nodeName, TryllNodeParamsBase @params,
                         Action<TryllError> onComplete = null);
public void Resume(string resumeNode = "", Action<TryllError> onComplete = null);
public TryllNodeParamsBase GetNodeParamsBaseline(string nodeName);
public void AppendInteractions(
    IReadOnlyList<TryllDialogInteraction> interactions,
    Action<uint, TryllError> onComplete = null);
public void RemoveInteractionsFromEnd(
    int count,
    Action<uint, TryllError> onComplete = null);
public void ExportDialog(
    Action<TryllDialogSnapshot, TryllError> onComplete = null);
public void ImportDialog(
    TryllDialogSnapshot snapshot,
    TryllDialogCompatibility compatibility = TryllDialogCompatibility.Strict,
    Action<TryllDialogImportResult, TryllError> onComplete = null);
public void SaveDialog(string path, string saveId,
                       Action<TryllDialogFileSaveResult, TryllError> onComplete = null);
public void LoadDialog(string path,
                       TryllDialogCompatibility compatibility = TryllDialogCompatibility.Strict,
                       Action<TryllDialogFileLoadResult, TryllError> onComplete = null);
public void RegisterTool(string toolName, Func<TryllToolCall, string> handler);
public void RegisterTool(string toolName, Func<TryllToolCall, Task<string>> handler);
public TryllAgentVariables Variables { get; }

// Delivery callbacks (last-writer-wins; pass null to clear).
// All fire on the Unity main thread. Register before SendMessage.
public void SetOnAnswerText(Action<string, string, bool, bool> callback);
public void SetOnTurnComplete(Action<TryllTurnStatus, string, int> callback);
public void SetOnError(Action<TryllError> callback);
public void SetOnTtsAudioFormat(Action<string, int, int, int> callback);
public void SetOnTtsAudio(Action<string, ArraySegment<short>> callback);
// Lip-sync track (GenerateAndSpeak with EmitVisemes). Additive multicast (+=);
// the array is valid only during the callback — copy it to keep it.
public event Action<string, TryllVisemeEvent[]> OnTtsVisemes;
public event Action<TryllError> ToolHandlerError;

// Lifetime
public void Dispose();
```

A code-created agent (`RequestCreateAgentAsync` without a component) must register `SetOnAnswerText` / `SetOnTurnComplete` before `SendMessage` or the reply is discarded. Pass `null` to clear. These replace any previously set callback (not multicast). `ToolHandlerError` is a separate multicast event for `RegisterTool` handler / resume failures.

`ChangeParams` / `onComplete` and `Resume` / `onComplete` deliver a [`TryllError`](#tryllerror) value — check `error.IsOk` (it is a struct; success is not null). `GetNodeParamsBaseline` returns a deep copy (or `null` if unknown). `ChangeParams` is allowed while idle **or paused**.

`Variables` is the agent's typed [Agent Variables](https://docs.tryllengine.com/v0.8/concepts/agent-variables.html) mirror — see [`TryllAgentVariables`](#tryllagentvariables) below.

For Inspector-friendly wrappers with `OnParamChanged` / `OnResumed` UnityEvents, prefer [`TryllAgentComponent`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-agent-component.html).

`AppendInteractions` / `RemoveInteractionsFromEnd` use [`TryllDialogInteraction`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/index.html#trylldialoginteraction) pairs — strict idle-only; see [Seed and edit dialog history](https://docs.tryllengine.com/v0.8/how-to/seed-and-edit-dialog-history.html).

`ExportDialog` / `ImportDialog` return a [`TryllDialogSnapshot`](https://docs.tryllengine.com/v0.8/how-to/export-and-import-dialog.html) (full-fidelity replace). `ToFlatBufferBytes` / `FromFlatBufferBytes` are the canonical `TDLG` helpers — the game owns the save file. See [Save and Load Agent State](https://docs.tryllengine.com/v0.8/how-to/save-and-load-agent-state.html) for multi-agent coordination, variable restoration, and save-slot handling.

Ownership

If you obtain a `TryllAgent` directly from `TryllClient.RequestCreateAgentAsync` (not via `TryllAgentComponent`), you own it and **must** call `Dispose()`.

______________________________________________________________________

## Experimental mutable knowledge bases

`TryllClient.EnsureMutableKbAsync(name, schema)` returns a session-bound `TryllMutableKb` handle. `LoadGameMemoryAsync` requires `Now` or `Lazy` — `Unspecified` fails locally with `8107 InvalidEmbedTiming`. Inspect stores in **Window ▸ Tryll ▸ Knowledge** (stores · Filter/Query records · modify).

```
public sealed class TryllMutableKb
{
    public bool IsValid { get; }
    public string Name { get; }
    public string Schema { get; }

    public Task<(List<TryllMutableKbWriteResult> results, TryllError error)> WriteAsync(
        IReadOnlyList<TryllMutableKbUpsert> records);
    public Task<(uint deletedCount, TryllError error)> DeleteAsync(IReadOnlyList<string> ids);
    public Task<(uint clearedCount, TryllError error)> ClearAsync();
    public Task<(List<TryllMutableKbHit> hits, TryllError error)> QueryAsync(
        string queryText, uint topK = 4, RetrievalMode retrievalMode = RetrievalMode.Dense);
    public Task<(TryllMutableKbRecord record, TryllError error)> GetAsync(string id);
    public Task<(TryllMutableKbListPage page, TryllError error)> ListAsync(
        uint offset = 0, uint limit = 50);
    public Task<(TryllMutableKbStats stats, TryllError error)> StatsAsync();
    public Task<(TryllMutableKbSnapshot snapshot, TryllError error)> ExportAsync();
    public Task<(TryllMutableKbImportResult result, TryllError error)> ImportAsync(
        TryllMutableKbSnapshot snapshot,
        TryllMutableKbCompatibility compatibility = TryllMutableKbCompatibility.Strict);
}
```

`ImportAsync` rejects a snapshot whose `Name` does not match the handle (`8105 MutableKbSnapshotInvalid`) before anything is sent.

______________________________________________________________________

## TryllDialogInteraction

One scripted user/assistant exchange for [`AppendInteractions`](#tryllagent).

```
[Serializable]
public struct TryllDialogInteraction
{
    public string UserMessage;      // empty → omit user side
    public string AssistantMessage; // empty → omit assistant side; both empty → skipped
}
```

______________________________________________________________________

## TryllAgentVariables

Typed per-agent [Variables](https://docs.tryllengine.com/v0.8/concepts/agent-variables.html) mirror, reachable via the `TryllAgent.Variables` property (e.g. `agentComp.Agent.Variables`).

```
public TryllError SetInt(string name, long value);
public TryllError SetFloat(string name, double value);
public TryllError SetString(string name, string value);
public TryllError SetBool(string name, bool value);
public TryllError SetStringSet(string name, IEnumerable<string> value);
public TryllError AddToSet(string name, string element);
public TryllError RemoveFromSet(string name, string element);
public TryllError Reset(string name);

public long   GetInt(string name);
public double GetFloat(string name);
public string GetString(string name);
public bool   GetBool(string name);
public IReadOnlyList<string> GetStringSet(string name);   // defensive copy
```

Setters validate locally — an unknown name or type mismatch returns a non-`Ok` `TryllError` immediately, with no wire round-trip. Writes update the local mirror synchronously; the actual `UpdateAgentVariablesRequest` is batched and flushed automatically immediately before the agent's next `SendMessage`/`Resume`/`ChangeParams` call.

```
var vars = agentComp.Agent.Variables;
var err = vars.SetInt("level", 13);
if (!err.IsOk) Debug.LogError($"{err.Code}: {err.Message}");
vars.AddToSet("quests_reached", "lost_amulet");
```

Declare variables on [`TryllWorkflowAsset.Variables`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-workflow-asset.html); a component's `VariableOverrides` may override an asset's declared initial value per-instance (Material-Instance pattern — the Inspector shows one optional row per declaration). Play Mode override edits update the live mirror; flush remains deferred (or use **Flush Now** in the Inspector).

______________________________________________________________________

## TryllError

Value-type error struct. Always check `IsOk` before using an API result.

```
[Serializable]
public struct TryllError
{
    public int    Code;        // 0 = OK. See TryllErrorCode enum for named values.
    public string Message;
    public bool   IsOk => Code == 0;
    public static TryllError Ok();
}
```

______________________________________________________________________

## TryllErrorCode

Named error code constants, generated from the server `ErrorCodes.h` into `Runtime/Generated/TryllErrorCode.gen.cs`. See [Error Codes](https://docs.tryllengine.com/v0.8/reference/error-codes.html) for the full table.

```
public enum TryllErrorCode
{
    Ok = 0,
    // Connection: 1001–1003
    // Session: 2001–2005 (2004 InvalidLocale, 2005 InvalidEngineSet)
    // Agent: 3001–3022 (3012 AgentNotPaused, 3013–3015 Variables,
    //        3016 InvalidToolResults, 3018–3022 Dialog snapshots)
    // Inference: 4001–4005 (4004–4005 KV cache)
    // VoiceInput / STT: 4100–4107
    // Protocol: 5001–5006 (5004 ProtocolVersionMismatch is raised client-side)
    // Download / models: 6001–6005, 6007–6009
    // StringStorage: 7001–7004
    // EmbeddedStringStorage: 8001–8003
    // Mutable KB: 8101–8109
}
```

______________________________________________________________________

## TryllGraphDescription and related types

```
[Serializable]
public class TryllGraphDescription
{
    public List<TryllNodeDescription> Nodes = new List<TryllNodeDescription>();
    public string                     StartNode;
    public string                     DefaultModelName;

    public TryllGraphDescription Clone();                  // deep copy, params included
    public T FindParams<T>(string nodeName) where T : TryllNodeParamsBase;
    public bool HasNode(string nodeName);
}

[Serializable]
public class TryllNodeDescription
{
    public string               Name;
    public TryllNodeParamsBase  Params;  // typed subclass; exit fields live here
}
```

Wiring between nodes is set as exit fields directly on the typed `Params` object — e.g. `TryllGenerateParams.DefaultExit`, `TryllRegexGuardrailParams.TriggeredExit`. An **empty string** routes to END (the default). See each node type's params class for the full list of exit fields and configuration params.

Tools are not on the node description. A `ToolCall` node carries them on `TryllToolCallParams.Tools` (`TryllToolDefinition[]`, each with `TryllToolParamDefinition` parameters).

Both types are classes, so one graph instance handed to several agents shares its params objects. Call `Clone()` before customising a copy per agent.

______________________________________________________________________

## TryllGraphBuilder

Fluent builder for `TryllGraphDescription`. Typed `AddXxx()` methods accept a typed params object; exit fields are set on that object before calling `AddXxx()`.

```
var graph = new TryllGraphBuilder()
    .AddGenerate("gen", new TryllGenerateParams
    {
        ModelName   = "mymodel",
        // DefaultExit defaults to "" (END)
    })
    .SetStartNode("gen")
    .SetDefaultModelName("mymodel")
    .Build();
```

______________________________________________________________________

## TryllModelInfo

```
[Serializable]
public struct TryllModelInfo
{
    public string                 Name;
    public TryllModelStatus       Status;
    public string                 HuggingFaceRepo;
    public long                   SizeBytes;
    public string                 ModelType;        // "language" | "embedding" | "stt" | "tts" | "vad"
    public string                 Engine;           // catalog engine, e.g. "llama-cpp", "sherpa-onnx"
    public string                 Audience;         // "public" | "experimental" | "internal" | ""
    public string[]               Languages;        // BCP-47 primary subtags; empty = undeclared
    public string                 ToolCallFormat;   // "chatml" | "llama3" | "mistral" | "generic" | ""
    public string                 ToolCallSupport;  // "supported" | "unsupported" | ""
    public int                    ContextSize;      // 0 = engine default
    public string[]               Files;
    public string                 LocalPath;        // empty when not on disk
    public bool                   HasDefaultSampling;
    public TryllSamplingParams    DefaultSampling;  // valid only when HasDefaultSampling
    public TryllModelDependency[] Dependencies;     // shared assets; empty when none
    public bool                   HasTtsCapabilities;
    public TryllTtsCapabilities   TtsCapabilities;  // valid only when HasTtsCapabilities
}
```

`Engine` is read-only catalog data. A model's variants share one engine, and no client call selects it.

______________________________________________________________________

## Enumerations

```
public enum TryllInferenceEngine : byte { Mock, LlamaCpp, OnnxGenAI, WindowsML, OpenVino, TensorRtLlm, SherpaOnnx, OnnxRuntime }
public enum TryllNodeType        : byte { Generate, RegexGuardrail, CannedResponse, ToolCall, Retrieve, Instruction, ClassifyIntent, IntentToInstruction, ClassifyIntentLLM, GenerateAndSpeak, Speak, Pause, Transform, Branch, WriteToMemory, Summarize }
public enum TryllTurnStatus      : byte { Success, Error, Cancelled, PauseTimedOut }
public enum TryllCancelMode      : byte { StopAndKeep, StopAndDiscard }
public enum TryllModelStatus     : byte { Absent, Local, Downloading, Loaded, Downloaded, Incomplete }
public enum TryllPlacement       : byte { BeforeUserAsUser, BeforeUserAsSystem, AfterUserAsUser, AfterUserAsSystem }
public enum TryllSendAnswer      : byte { None, Whole, Streamed }
public enum TryllHistoryRole     : byte { None, Assistant }

// Namespace Tryll (not Tryll.Client) — add `using Tryll;`.
public enum WireViseme : byte { sil, PP, FF, TH, DD, kk, CH, SS, nn, RR, aa, E, ih, oh, ou }
```

`TryllNodeType` is 0-based, while the wire `NodeParams` union tag is 1-based. Use the named value, never a raw number. The per-node option enums (`TryllToolCallMode`, `TryllToolCallDisposition`, `TryllRetrievalMode`, `TryllBranchTest`, and the rest) live in `TryllGraphDescription.cs` next to these.

______________________________________________________________________

## Threading and error handling summary

- All events and `Task<T>` completions fire on the Unity **main thread**.
- Always check `error.IsOk` before using any `Task<T>` result.
- Call `TryllAgentComponent.SendMessage` / `CreateAgent` from the main thread only.
- Use `await` only from `async void` methods rooted on the main thread (e.g., `async void Start()`).
