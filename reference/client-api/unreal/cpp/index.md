# Unreal C++ API Reference

Full API reference for the `TryllClient` Unreal Engine plugin, auto-generated from Doxygen documentation blocks in `tryll/clients/unreal/Source/TryllClient/Public/`. Browse individual classes and structs in the sidebar.

## Entry points

| Type                   | Role                                                                                                                             |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `UTryllSubsystem`      | `UGameInstanceSubsystem` — connect, configure, manage models and storages                                                        |
| `UTryllWorkflowAsset`  | `UDataAsset` — editor-authored graph; assign to an agent component                                                               |
| `FTryllAgent`          | Agent handle returned by `UTryllSubsystem::CreateAgent`                                                                          |
| `FTryllAgentVariables` | Per-agent [Variables](https://docs.tryllengine.com/concepts/agent-variables.html) mirror — typed setters/getters, deferred flush |

## Components

`UActorComponent`s you add to an actor. Their Blueprint surface is in the [Blueprint Catalog](https://docs.tryllengine.com/reference/client-api/unreal/blueprint-catalog.html); per-member detail is on each class page below.

| Type                                                                                                              | Role                                                            |
| ----------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| [`UTryllAgentComponent`](https://docs.tryllengine.com/tryll-unreal/class_u_tryll_agent_component.html)            | Manages one agent — graph, `SendMessage`, runtime param changes |
| [`UTryllSpeakerComponent`](https://docs.tryllengine.com/tryll-unreal/class_u_tryll_speaker_component.html)        | Plays streaming TTS audio                                       |
| [`UTryllVoiceInputComponent`](https://docs.tryllengine.com/tryll-unreal/class_u_tryll_voice_input_component.html) | Microphone capture + STT                                        |

## Graph types

| Type                     | Role                                                                         |
| ------------------------ | ---------------------------------------------------------------------------- |
| `FTryllGraphDescription` | Complete graph: nodes (with typed params carrying exit fields), start id     |
| `FTryllGraphBuilder`     | Fluent builder for `FTryllGraphDescription`                                  |
| `FTryllNodeDescription`  | Single node definition — name + typed params pointer                         |
| `UTryllNodeParamsBase`   | Base class for typed node params; exit fields live on each concrete subclass |

## Supporting types

| Type                                                                                                     | Role                                                                                                        |
| -------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `FTryllError`                                                                                            | Error envelope — `ETryllErrorCode` + message string                                                         |
| `FTryllModelInfo`                                                                                        | Model catalog row returned by `ListModels`                                                                  |
| `FTryllToolDefinition`                                                                                   | Tool declaration                                                                                            |
| `FTryllToolParamDefinition`                                                                              | Single parameter within a `FTryllToolDefinition`                                                            |
| `FTryllVariableDecl`                                                                                     | Name + typed initial value, declared on `UTryllWorkflowAsset::Variables` / passed to `CreateAgent`          |
| `FTryllVariableOverride`                                                                                 | Per-instance initial-value override for a declared variable (Material-Instance pattern)                     |
| `UTryllSubsystem::FEmbeddedStorageInfo`                                                                  | Embedded-storage descriptor                                                                                 |
| [`UTryllRuntimeSettings`](https://docs.tryllengine.com/tryll-unreal/class_u_tryll_runtime_settings.html) | Project settings (`UDeveloperSettings`) — engines, auto-launch, storage                                     |
| `UTryllNodeParamsFactory`                                                                                | Blueprint/C++ factory: `MakeXxxParams` + `CloneParams` for node params                                      |
| `FTryllVoiceInput`                                                                                       | STT session handle owned by `UTryllVoiceInputComponent`                                                     |
| `FTryllTranscriptUpdate`                                                                                 | Transcript update payload (`Text`, `Kind`, `AudioMsConsumed`)                                               |
| `ETryllWorkload`                                                                                         | `Interactive` / `Background` — scheduling hint on `UTryllAgentComponent::Workload` and `RequestCreateAgent` |

## Headers

| Header                               | Declares                                                                                                           |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| `TryllSubsystem.h`                   | `UTryllSubsystem`, `FEmbeddedStorageInfo`                                                                          |
| `TryllAgent.h`                       | `FTryllAgent`, `FTryllDialogInteraction`                                                                           |
| `TryllAgentVariables.h`              | `FTryllAgentVariables`                                                                                             |
| `TryllVariableDecl.h`                | `FTryllVariableDecl`, `FTryllVariableOverride`, `ETryllVariableType`                                               |
| `TryllAgentComponent.h`              | `UTryllAgentComponent`                                                                                             |
| `TryllSpeakerComponent.h`            | `UTryllSpeakerComponent`                                                                                           |
| `TryllVoiceInputComponent.h`         | `UTryllVoiceInputComponent`                                                                                        |
| `TryllVoiceInput.h`                  | `FTryllVoiceInput`, `FTryllTranscriptUpdate`, `ETryllTranscriptUpdateKind`                                         |
| `TryllWorkflowAsset.h`               | `UTryllWorkflowAsset`                                                                                              |
| `TryllRuntimeSettings.h`             | `UTryllRuntimeSettings`, `EServerBuildVariant`                                                                     |
| `TryllGraphDescription.h`            | `FTryllGraphDescription`, `FTryllGraphBuilder`, `FTryllNodeDescription`, `ETryllInferenceEngine`, `ETryllNodeType` |
| `Generated/TryllNodeParamsFactory.h` | `UTryllNodeParamsFactory`                                                                                          |
| `TryllModelInfo.h`                   | `FTryllModelInfo`, `ETryllModelStatus`                                                                             |
| `TryllError.h`                       | `FTryllError`, `ETryllErrorCode`                                                                                   |
| `TryllThrottle.h`                    | `ETryllWorkload`                                                                                                   |

## Selected method signatures

### `UTryllSubsystem::CreateSession`

```
UFUNCTION(BlueprintCallable, Category = "Tryll|Session")
void CreateSession(
    ETryllInferenceEngine Engine,
    FString               GameName          = TEXT(""),
    ETryllInferenceEngine SttEngine         = ETryllInferenceEngine::Mock,
    ETryllInferenceEngine TtsEngine         = ETryllInferenceEngine::Mock,
    ETryllInferenceEngine EmbeddingEngine   = ETryllInferenceEngine::Mock,
    FString               StorageDataFolder = TEXT(""));
```

`GameName` and `StorageDataFolder` map to the same-named `UTryllRuntimeSettings` fields. `CreateAgent`, `CreateEmbeddedStringStorage`, and `CreateVoiceInput` fail fast if a referenced model is not already on disk — acquire models explicitly beforehand via the editor Model Manager.

To configure from the project settings asset instead of passing parameters, call `CreateSessionFromSettings()` — this runs automatically once connected when `UTryllRuntimeSettings::bAutoCreateSession` is `true`.

## Usage

### Build a graph in C++

Set node parameters on a params object, then add it to the builder and create the agent from the resulting graph:

```
UTryllGenerateParams* P = UTryllNodeParamsFactory::MakeGenerateParams(this);
P->bOverrideModelName    = true;
P->ModelName             = TEXT("qwen2.5-0.5b-instruct");
P->bOverrideSystemPrompt = true;
P->SystemPrompt          = TEXT("You are helpful.");

const FTryllGraphDescription Graph =
    FTryllGraphBuilder()
        .AddNode(TEXT("gen"), P)
        .SetStartNode(TEXT("gen"))
        .SetDefaultModelName(TEXT("qwen2.5-0.5b-instruct"))
        .Build();

Subsystem->RequestCreateAgent(Graph,
    [](TSharedPtr<FTryllAgent> Agent, FTryllError Error) { /* … */ });
```

The typed `AddXxx` helpers (`AddGenerate`, `AddRetrieve`, …) are generated into `TryllGraphBuilder.Nodes.h`; `AddNode(Name, Params)` accepts any node type.

Inline graphs are temporary

`UTryllAgentComponent::InlineGraphDescription` — assigning a code-built graph to the component instead of using a Workflow Asset — is a temporary feature and will be removed in a future release. When you drive an agent through `UTryllAgentComponent`, author the graph as a `UTryllWorkflowAsset` (Content Browser → **Tryll → Workflow Asset**) and assign it to the component's **Workflow Asset** slot.

### Change a node parameter at runtime

Clone the baseline, set the field's `bOverrideXxx` flag along with the value, then send. The agent must be idle **or paused**; `OnParamChanged` fires on completion:

```
auto* Base = Cast<UTryllGenerateParams>(AgentComponent->GetNodeParamsBaseline(TEXT("gen")));
auto* Mut  = Cast<UTryllGenerateParams>(UTryllNodeParamsFactory::CloneParams(Base, this));
Mut->bOverrideSystemPrompt = true;
Mut->SystemPrompt          = TEXT("New prompt");
AgentComponent->ChangeParams(TEXT("gen"), Mut);
```

### Pause and resume a turn

`FTryllAgent::Resume` / `UTryllAgentComponent::Resume` continue a turn suspended at a [Pause node](https://docs.tryllengine.com/reference/nodes/pause.html) or a paused `ToolCall`. Bind `UTryllSubsystem::OnPaused` (Blueprint `On Paused`) to react to the pause — it lives on the **subsystem**, not the component (the component exposes `OnResumed`, not `OnPaused`). `ChangeAgentParam` is allowed while parked:

```
void AThisClass::HandlePaused(int64 AgentId, const FString& NodeName, const FString& PendingExit)
{
    // ChangeParams here is allowed — the agent is paused, not busy.
    AgentComponent->Resume(); // empty = continue via PendingExit; or Resume(TEXT("other_node")) to jump
}
```

See [How to pause and resume a turn](https://docs.tryllengine.com/how-to/pause-and-resume-turns.html).

### Append or remove scripted dialog history

`FTryllAgent::AppendInteractions` / `RemoveInteractionsFromEnd` mutate history without running the graph. **Strict idle-only** (`AgentBusy` while running, paused, or during a KV-cache operation). Each `FTryllDialogInteraction` has `UserMessage` / `AssistantMessage` — empty strings omit that side; both empty → skipped server-side.

```
TArray<FTryllDialogInteraction> Seed;
{
    FTryllDialogInteraction Opener;
    Opener.AssistantMessage = TEXT("Welcome, traveler.");
    Seed.Add(Opener);
}
Agent->AppendInteractions(Seed,
    [](uint32 Count, const FTryllError& Err)
    {
        if (Err.IsOk()) UE_LOG(LogTemp, Log, TEXT("Appended %u interactions"), Count);
    });

Agent->RemoveInteractionsFromEnd(1); // drop last whole interaction
```

See [Seed and edit dialog history](https://docs.tryllengine.com/how-to/seed-and-edit-dialog-history.html).

### Throttle the GPU and mark background agents

`UTryllSubsystem::SetInferenceThrottle` (Blueprint-callable, `Tryll|Session`) reports how hard the server should yield the GPU back to your game — `0.f` = full speed (the default for a session that never calls it), `1.f` = maximum yielding. Fire-and-forget: no request id, no response, no completion delegate, so calling it from `Tick` costs nothing but the frame. It never changes what is generated, only how fast.

```
void AMyGameMode::Tick(float DeltaSeconds)
{
    Super::Tick(DeltaSeconds);
    // Example policy: yield harder as your own frame time worsens.
    const float Pressure = FMath::GetMappedRangeValueClamped(
        FVector2D(11.f, 20.f), FVector2D(0.f, 1.f), DeltaSeconds * 1000.f);
    GetGameInstance()->GetSubsystem<UTryllSubsystem>()->SetInferenceThrottle(Pressure);
}
```

Agents nobody is waiting on should say so, so their inference is the first to give way. On `UTryllAgentComponent` this is the **Workload** property (`Tryll|Agent` category, editable in the Details panel); in C++ it is the trailing `Workload` argument to `UTryllSubsystem::RequestCreateAgent`:

```
Subsystem->RequestCreateAgent(
    Graph,
    [](TSharedPtr<FTryllAgent> Agent, const FTryllError& Err) { /* ... */ },
    /*bEnableDiagnostics=*/false,
    /*bMaintainDialogueHistory=*/true,
    /*Variables=*/{},
    ETryllKvCacheInitialization::AllocateOnly,
    ETryllWorkload::Background);
```

Everything else the server classifies from the graph itself, per turn — streamed `Generate` text is protected, spoken and buffered work absorbs the slack. See [Wire Protocol](https://docs.tryllengine.com/reference/wire-protocol.html#inference-throttle-and-agent-workload) and [Server configuration](https://docs.tryllengine.com/reference/server-configuration.html#inference-scheduler).

### Read and write Agent Variables

`FTryllAgent::GetVariables()` returns the [Agent Variables](https://docs.tryllengine.com/concepts/agent-variables.html) mirror. Writes update the local mirror synchronously; the wire update is batched and flushed automatically immediately before the next `SendMessage`/`Resume`/`ChangeParams` call:

```
FTryllAgentVariables& Vars = Agent->GetVariables();
FTryllError Err = Vars.SetInt(TEXT("level"), 13);
if (!Err.IsOk()) UE_LOG(LogTemp, Warning, TEXT("%d: %s"), Err.Code, *Err.Message);
Vars.AddToSet(TEXT("quests_reached"), TEXT("lost_amulet"));
// Optional eager flush (Details "Flush Now" / UTryllAgentVariables::FlushIfDirty):
Vars.FlushIfDirty([](const FTryllError& FlushErr) { /* ... */ });
```

An unknown name or type mismatch returns a non-OK `FTryllError` immediately (no wire round-trip), using the same `3013`/`3014` codes the server would return.

Per-instance **Variable Overrides** on `UTryllAgentComponent` use the Material-Instance pattern (see [Agent Variables](https://docs.tryllengine.com/concepts/agent-variables.html)). Details edits live-apply through `UTryllAgentComponent::GetLiveVariables()` when a PIE agent or Tryll Chat editor-preview agent is attached. Unchecking an override assigns the workflow declaration default. Helpers live in `TryllVariableOverrideUtils`.

Override flags

Every value field on a `UTryllXxxParams` is gated by a `bOverrideXxx` flag. Set the flag or the value is silently ignored — the node falls back to the graph's default model / authored defaults. The clone preserves the baseline's flags, but set the flag explicitly when you start overriding a field the node didn't author.

Note

The component classes (`UTryllAgentComponent`, `UTryllSpeakerComponent`, `UTryllVoiceInputComponent`) and project settings (`UTryllRuntimeSettings`) are also documented in the [Blueprint Catalog](https://docs.tryllengine.com/reference/client-api/unreal/blueprint-catalog.html); per-member detail is on the auto-generated class pages in the sidebar.
