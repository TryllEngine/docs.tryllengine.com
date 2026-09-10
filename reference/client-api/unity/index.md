# Unity Client API

**Package:** `com.tryll.client`\
**Namespace:** `Tryll.Client`\
**Target:** Unity 6 (6000.x), .NET Standard 2.1, Windows Standalone + Editor

> The Unity client is a hand-authored C# port of the Unreal plugin. The public surface is documented on this page and the entity pages linked below. Internal types under `Tryll.Client.Internal` are not part of the public API.

______________________________________________________________________

## Inspector entities

These types appear in the Inspector and have dedicated reference pages.

| Type                                                                                                                 | Kind               | Description                                                                                                           |
| -------------------------------------------------------------------------------------------------------------------- | ------------------ | --------------------------------------------------------------------------------------------------------------------- |
| [TryllClient](https://docs.tryllengine.com/reference/client-api/unity/tryll-client.html)                             | `MonoBehaviour`    | Singleton session manager. Created automatically — do not add it to a scene.                                          |
| [TryllAgentComponent](https://docs.tryllengine.com/reference/client-api/unity/tryll-agent-component.html)            | `MonoBehaviour`    | Drop-in component that owns and drives one agent on a GameObject.                                                     |
| [TryllSpeaker](https://docs.tryllengine.com/reference/client-api/unity/tryll-speaker.html)                           | `MonoBehaviour`    | Plays streaming TTS audio from an agent through an `AudioSource`. Assign to a `TryllAgentComponent`'s `Speaker` slot. |
| [TryllVoiceInputComponent](https://docs.tryllengine.com/reference/client-api/unity/tryll-voice-input-component.html) | `MonoBehaviour`    | Microphone capture component that streams audio to the STT pipeline.                                                  |
| [TryllWorkflowAsset](https://docs.tryllengine.com/reference/client-api/unity/tryll-workflow-asset.html)              | `ScriptableObject` | Project asset that stores a `TryllGraphDescription`.                                                                  |
| [TryllRuntimeSettings](https://docs.tryllengine.com/reference/client-api/unity/tryll-runtime-settings.html)          | `ScriptableObject` | Package settings (server path, host, port). Edit via **Project Settings → Tryll Client**.                             |

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
public TryllAgentVariables Variables { get; }

// Lifetime
public void Dispose();
```

`ChangeParams` / `onComplete` and `Resume` / `onComplete` deliver a [`TryllError`](#tryllerror) value — check `error.IsOk` (it is a struct; success is not null). `GetNodeParamsBaseline` returns a deep copy (or `null` if unknown). `ChangeParams` is allowed while idle **or paused**.

`Variables` is the agent's typed [Agent Variables](https://docs.tryllengine.com/concepts/agent-variables.html) mirror — see [`TryllAgentVariables`](#tryllagentvariables) below.

For Inspector-friendly wrappers with `OnParamChanged` / `OnResumed` UnityEvents, prefer [`TryllAgentComponent`](https://docs.tryllengine.com/reference/client-api/unity/tryll-agent-component.html).

`AppendInteractions` / `RemoveInteractionsFromEnd` use [`TryllDialogInteraction`](https://docs.tryllengine.com/reference/client-api/unity/index.html#trylldialoginteraction) pairs — strict idle-only; see [Seed and edit dialog history](https://docs.tryllengine.com/how-to/seed-and-edit-dialog-history.html).

`ExportDialog` / `ImportDialog` return a [`TryllDialogSnapshot`](https://docs.tryllengine.com/how-to/export-and-import-dialog.html) (full-fidelity replace). `ToFlatBufferBytes` / `FromFlatBufferBytes` are the canonical `TDLG` helpers — the game owns the save file. See [Save and Load Agent State](https://docs.tryllengine.com/how-to/save-and-load-agent-state.html) for multi-agent coordination, variable restoration, and save-slot handling.

Ownership

If you obtain a `TryllAgent` directly from `TryllClient.RequestCreateAgentAsync` (not via `TryllAgentComponent`), you own it and **must** call `Dispose()`.

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

Typed per-agent [Variables](https://docs.tryllengine.com/concepts/agent-variables.html) mirror, reachable via the `TryllAgent.Variables` property (e.g. `agentComp.Agent.Variables`).

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

Declare variables on [`TryllWorkflowAsset.Variables`](https://docs.tryllengine.com/reference/client-api/unity/tryll-workflow-asset.html); a component's `VariableOverrides` may override an asset's declared initial value per-instance (Material-Instance pattern — the Inspector shows one optional row per declaration). Play Mode override edits update the live mirror; flush remains deferred (or use **Flush Now** in the Inspector).

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

Named error code constants. Values match the server `ErrorCodes.h`. See [Error Codes](https://docs.tryllengine.com/reference/error-codes.html) for the full table.

```
public enum TryllErrorCode
{
    Ok = 0,
    // Connection: 1001–1003
    // Session: 2001–2003
    // Agent: 3001–3007, 3012 (AgentNotPaused), 3013–3015 (Variables)
    // Inference: 4001–4003
    // VoiceInput / STT: 4100–4105
    // Protocol: 5001–5004
    // Download: 6001–6004
    // StringStorage: 7001–7003
}
```

______________________________________________________________________

## TryllGraphDescription and related types

```
[Serializable]
public struct TryllGraphDescription
{
    public List<TryllNodeDescription> Nodes;
    public string                     StartNode;
    public string                     DefaultModelName;
}

[Serializable]
public struct TryllNodeDescription
{
    public string               Name;
    public TryllNodeParamsBase  Params;  // typed subclass; exit fields live here
    public List<TryllToolDefinition> Tools;
}
```

Wiring between nodes is set as exit fields directly on the typed `Params` object — e.g. `TryllGenerateParams.DefaultExit`, `TryllRegexGuardrailParams.TriggeredExit`. An **empty string** routes to END (the default). See each node type's params class for the full list of exit fields and configuration params.

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
    public string           Name;
    public TryllModelStatus Status;
    public string           HuggingFaceRepo;
    public long             SizeBytes;
}
```

______________________________________________________________________

## Enumerations

```
public enum TryllInferenceEngine : byte { Mock, LlamaCpp, OnnxGenAI, WindowsML, OpenVino, TensorRtLlm, SherpaOnnx }
public enum TryllNodeType        : byte { Generate, RegexGuardrail, CannedResponse, ToolCall, Retrieve, Instruction, ClassifyIntent, IntentToInstruction, ClassifyIntentLLM, GenerateAndSpeak, Speak, Pause, Transform }
public enum TryllTurnStatus      : byte { Success, Error, Cancelled }
public enum TryllModelStatus     : byte { Absent, Local, Downloading, Loaded, Downloaded }
public enum TryllPlacement       : byte { BeforeUserAsUser, BeforeUserAsSystem, AfterUserAsUser, AfterUserAsSystem }
public enum TryllSendAnswer      : byte { None, Whole, Streamed }
public enum TryllHistoryRole     : byte { None, Assistant }
public enum TryllKnowledgeAllEmptyBehavior : byte { UseAlternateTemplate, Skip }
```

______________________________________________________________________

## Threading and error handling summary

- All events and `Task<T>` completions fire on the Unity **main thread**.
- Always check `error.IsOk` before using any `Task<T>` result.
- Call `TryllAgentComponent.SendMessage` / `CreateAgent` from the main thread only.
- Use `await` only from `async void` methods rooted on the main thread (e.g., `async void Start()`).
