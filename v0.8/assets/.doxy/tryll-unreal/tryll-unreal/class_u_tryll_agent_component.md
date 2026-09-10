

# Class UTryllAgentComponent



[**ClassList**](annotated.md) **>** [**UTryllAgentComponent**](class_u_tryll_agent_component.md)



[More...](#detailed-description)

* `#include <TryllAgentComponent.h>`



Inherits the following classes: UActorComponent


















## Public Attributes

| Type | Name |
| ---: | :--- |
|  ETryllKvCacheInitialization | [**KvCacheInitialization**](#variable-kvcacheinitialization)   = `ETryllKvCacheInitialization::AllocateOnly`<br> |
|  FOnTryllAgentReady | [**OnAgentReady**](#variable-onagentready)  <br> |
|  FOnTryllAgentAnswerFull | [**OnAnswerFull**](#variable-onanswerfull)  <br> |
|  FOnTryllAgentAnswerText | [**OnAnswerText**](#variable-onanswertext)  <br> |
|  FOnTryllAppendInteractionsError | [**OnAppendInteractionsError**](#variable-onappendinteractionserror)  <br> |
|  FOnTryllDialogExportedNative | [**OnDialogExported**](#variable-ondialogexported)  <br> |
|  FOnTryllDialogImportedNative | [**OnDialogImported**](#variable-ondialogimported)  <br> |
|  FOnTryllAgentError | [**OnError**](#variable-onerror)  <br> |
|  FOnTryllInteractionsAppended | [**OnInteractionsAppended**](#variable-oninteractionsappended)  <br> |
|  FOnTryllInteractionsRemoved | [**OnInteractionsRemoved**](#variable-oninteractionsremoved)  <br> |
|  FOnTryllKvCacheEvictError | [**OnKvCacheEvictError**](#variable-onkvcacheevicterror)  <br> |
|  FOnTryllKvCacheEvicted | [**OnKvCacheEvicted**](#variable-onkvcacheevicted)  <br> |
|  FOnTryllKvCachePrefillError | [**OnKvCachePrefillError**](#variable-onkvcacheprefillerror)  <br> |
|  FOnTryllKvCachePrefilled | [**OnKvCachePrefilled**](#variable-onkvcacheprefilled)  <br> |
|  FOnTryllKvCacheStatus | [**OnKvCacheStatus**](#variable-onkvcachestatus)  <br> |
|  FOnTryllKvCacheStatusError | [**OnKvCacheStatusError**](#variable-onkvcachestatuserror)  <br> |
|  FOnTryllMessageQueued | [**OnMessageQueued**](#variable-onmessagequeued)  <br> |
|  FOnTryllParamChanged | [**OnParamChanged**](#variable-onparamchanged)  <br> |
|  FOnTryllRemoveInteractionsError | [**OnRemoveInteractionsError**](#variable-onremoveinteractionserror)  <br> |
|  FOnTryllResumed | [**OnResumed**](#variable-onresumed)  <br> |
|  FOnTryllAgentTurnComplete | [**OnTurnComplete**](#variable-onturncomplete)  <br> |
|  TObjectPtr&lt; [**UTryllSpeakerComponent**](class_u_tryll_speaker_component.md) &gt; | [**Speaker**](#variable-speaker)   = `nullptr`<br> |
|  TArray&lt; [**FTryllVariableOverride**](struct_f_tryll_variable_override.md) &gt; | [**VariableOverrides**](#variable-variableoverrides)  <br> |
|  [**UTryllWorkflowAsset**](class_u_tryll_workflow_asset.md) \* | [**WorkflowAsset**](#variable-workflowasset)   = `nullptr`<br> |
|  ETryllWorkload | [**Workload**](#variable-workload)   = `ETryllWorkload::Interactive`<br> |
|  bool | [**bAutoCreateOnConnect**](#variable-bautocreateonconnect)   = `true`<br> |
|  bool | [**bEnableDiagnostics**](#variable-benablediagnostics)   = `false`<br> |
|  bool | [**bMaintainDialogueHistory**](#variable-bmaintaindialoguehistory)   = `true`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  void | [**AppendInteractions**](#function-appendinteractions) (const TArray&lt; [**FTryllDialogInteraction**](struct_f_tryll_dialog_interaction.md) &gt; & Interactions) <br> |
|  void | [**Cancel**](#function-cancel) (ETryllCancelMode Mode=ETryllCancelMode::StopAndKeep) <br> |
|  void | [**ChangeParams**](#function-changeparams) (const FString & NodeName, [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* Params) <br> |
|  void | [**CreateAgent**](#function-createagent) () <br> |
|  void | [**DestroyAgent**](#function-destroyagent) () <br> |
|  void | [**EvictKvCache**](#function-evictkvcache) () <br> |
|  void | [**ExportDialog**](#function-exportdialog) () <br> |
|  const FTryllToolHandler \* | [**FindToolHandler**](#function-findtoolhandler) (const FString & ToolName) const<br> |
|  const TSharedPtr&lt; [**FTryllAgent**](class_f_tryll_agent.md) &gt; & | [**GetAgent**](#function-getagent) () const<br> |
|  int64 | [**GetAgentId**](#function-getagentid) () const<br> |
|  void | [**GetKvCacheStatus**](#function-getkvcachestatus) () <br> |
|  [**FTryllAgentVariables**](class_f_tryll_agent_variables.md) \* | [**GetLiveVariables**](#function-getlivevariables) () <br> |
|  [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* | [**GetNodeParamsBaseline**](#function-getnodeparamsbaseline) (const FString & NodeName) const<br> |
|  FString | [**GetPausedNode**](#function-getpausednode) () const<br> |
|  const FString & | [**GetPendingMessage**](#function-getpendingmessage) () const<br> |
|  [**UTryllAgentVariables**](class_u_tryll_agent_variables.md) \* | [**GetVariables**](#function-getvariables) () <br> |
|  bool | [**HasLiveAgent**](#function-hasliveagent) () const<br> |
|  bool | [**HasPausedToolBatch**](#function-haspausedtoolbatch) () const<br> |
|  bool | [**HasPendingMessage**](#function-haspendingmessage) () const<br> |
|  void | [**ImportDialog**](#function-importdialog) (const [**FTryllDialogSnapshot**](struct_f_tryll_dialog_snapshot.md) & Snapshot, ETryllDialogCompatibility Compatibility=ETryllDialogCompatibility::Strict) <br> |
|  bool | [**IsAgentReady**](#function-isagentready) () const<br> |
|  void | [**PrefillKvCache**](#function-prefillkvcache) () <br> |
|  void | [**RegisterTool**](#function-registertool-12) (const FString & ToolName, FTryllToolHandler Handler) <br> |
|  void | [**RegisterTool**](#function-registertool-22) (const FString & ToolName, FTryllSimpleToolHandler Handler) <br> |
|  void | [**RemoveInteractionsFromEnd**](#function-removeinteractionsfromend) (int32 Count) <br> |
|  void | [**Resume**](#function-resume) (const FString & ResumeNode=TEXT("")) <br> |
|  void | [**ResumeWithToolResult**](#function-resumewithtoolresult) (const [**FTryllToolResult**](struct_f_tryll_tool_result.md) & ToolResult, const FString & ResumeNode=TEXT("")) <br> |
|  void | [**ResumeWithToolResults**](#function-resumewithtoolresults) (const TArray&lt; [**FTryllToolResult**](struct_f_tryll_tool_result.md) &gt; & ToolResults, const FString & ResumeNode=TEXT("")) <br> |
|  void | [**RetryPausedToolBatch**](#function-retrypausedtoolbatch) () <br> |
|  void | [**SendMessage**](#function-sendmessage) (const FString & Message) <br> |
|  bool | [**TryFlushPendingMessage**](#function-tryflushpendingmessage) () <br> |
|   | [**UTryllAgentComponent**](#function-utryllagentcomponent) () <br> |
|  bool | [**UnregisterTool**](#function-unregistertool) (const FString & ToolName) <br> |
























## Protected Functions

| Type | Name |
| ---: | :--- |
| virtual void | [**BeginPlay**](#function-beginplay) () override<br> |
| virtual void | [**EndPlay**](#function-endplay) (const EEndPlayReason::Type EndPlayReason) override<br> |




## Detailed Description


Actor component: manages one Tryll server-side agent. Attach to any Actor; Blueprint users bind to the delegates and call [**SendMessage()**](class_u_tryll_agent_component.md#function-sendmessage).


The graph comes from WorkflowAsset — a designer-authored data asset in the Content Browser. CreateAgent fails if no asset is assigned. To build a graph in C++ without an asset, use [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) and [**UTryllSubsystem::RequestCreateAgent**](class_u_tryll_subsystem.md#function-requestcreateagent).


The component auto-creates an agent once the session is ready (if bAutoCreateOnConnect) — it queues until OnCreateSessionComplete if it spawns before the session is configured. It auto-destroys the agent on EndPlay. 


    
## Public Attributes Documentation




### variable KvCacheInitialization 

```C++
ETryllKvCacheInitialization UTryllAgentComponent::KvCacheInitialization;
```



How language-model KV contexts are materialised at agent creation. 


        

<hr>



### variable OnAgentReady 

```C++
FOnTryllAgentReady UTryllAgentComponent::OnAgentReady;
```



Fired once after the server confirms the agent has been created. 


        

<hr>



### variable OnAnswerFull 

```C++
FOnTryllAgentAnswerFull UTryllAgentComponent::OnAnswerFull;
```



Fired after TurnComplete with the full accumulated response text. 


        

<hr>



### variable OnAnswerText 

```C++
FOnTryllAgentAnswerText UTryllAgentComponent::OnAnswerText;
```



Fired for each streaming AnswerText chunk.
* Text = the chunk (delta or accumulated, per server config).
* bIsFinal = true on the last chunk before TurnComplete. 




        

<hr>



### variable OnAppendInteractionsError 

```C++
FOnTryllAppendInteractionsError UTryllAgentComponent::OnAppendInteractionsError;
```




<hr>



### variable OnDialogExported 

```C++
FOnTryllDialogExportedNative UTryllAgentComponent::OnDialogExported;
```




<hr>



### variable OnDialogImported 

```C++
FOnTryllDialogImportedNative UTryllAgentComponent::OnDialogImported;
```




<hr>



### variable OnError 

```C++
FOnTryllAgentError UTryllAgentComponent::OnError;
```



Fired on any agent-level error (send failure, server error response). 


        

<hr>



### variable OnInteractionsAppended 

```C++
FOnTryllInteractionsAppended UTryllAgentComponent::OnInteractionsAppended;
```




<hr>



### variable OnInteractionsRemoved 

```C++
FOnTryllInteractionsRemoved UTryllAgentComponent::OnInteractionsRemoved;
```




<hr>



### variable OnKvCacheEvictError 

```C++
FOnTryllKvCacheEvictError UTryllAgentComponent::OnKvCacheEvictError;
```




<hr>



### variable OnKvCacheEvicted 

```C++
FOnTryllKvCacheEvicted UTryllAgentComponent::OnKvCacheEvicted;
```




<hr>



### variable OnKvCachePrefillError 

```C++
FOnTryllKvCachePrefillError UTryllAgentComponent::OnKvCachePrefillError;
```




<hr>



### variable OnKvCachePrefilled 

```C++
FOnTryllKvCachePrefilled UTryllAgentComponent::OnKvCachePrefilled;
```




<hr>



### variable OnKvCacheStatus 

```C++
FOnTryllKvCacheStatus UTryllAgentComponent::OnKvCacheStatus;
```




<hr>



### variable OnKvCacheStatusError 

```C++
FOnTryllKvCacheStatusError UTryllAgentComponent::OnKvCacheStatusError;
```




<hr>



### variable OnMessageQueued 

```C++
FOnTryllMessageQueued UTryllAgentComponent::OnMessageQueued;
```



Fired when SendMessage could not hand the message over immediately and queued it instead. Carries the queued text, so a game can show that it is waiting rather than leaving the player wondering whether their line registered. 


        

<hr>



### variable OnParamChanged 

```C++
FOnTryllParamChanged UTryllAgentComponent::OnParamChanged;
```



Fired when a ChangeParam request completes.
* bSuccess = true on Ack; false on any server error.
* ErrorMessage = empty on success; human-readable description on failure. 




        

<hr>



### variable OnRemoveInteractionsError 

```C++
FOnTryllRemoveInteractionsError UTryllAgentComponent::OnRemoveInteractionsError;
```




<hr>



### variable OnResumed 

```C++
FOnTryllResumed UTryllAgentComponent::OnResumed;
```



Fired when a Resume request completes.
* bSuccess = true on Ack; false on any server error (e.g. AgentNotPaused, UnknownNode).
* ErrorMessage = empty on success; human-readable description on failure. 




        

<hr>



### variable OnTurnComplete 

```C++
FOnTryllAgentTurnComplete UTryllAgentComponent::OnTurnComplete;
```



Fired when TurnComplete arrives, carrying the turn outcome status. 


        

<hr>



### variable Speaker 

```C++
TObjectPtr<UTryllSpeakerComponent> UTryllAgentComponent::Speaker;
```



Optional speaker component used to play streaming TTS audio frames emitted by the agent (e.g. from a GenerateAndSpeak node).


If left unset, BeginPlay will auto-discover a [**UTryllSpeakerComponent**](class_u_tryll_speaker_component.md) on the same actor (the common case — add both components to your actor and you're done). Set this explicitly when the owning actor has more than one speaker and you need to pick a specific one.


To disable TTS playback entirely: leave this unset AND have no speaker component on the actor. 


        

<hr>



### variable VariableOverrides 

```C++
TArray<FTryllVariableOverride> UTryllAgentComponent::VariableOverrides;
```



Per-instance value overrides for the workflow's declared variables (Material-Instance pattern — name/type come from the declaration; only the initial value can be overridden). Entries whose Name/Type no longer match a declared variable are skipped with a warning at agent-creation time. 


        

<hr>



### variable WorkflowAsset 

```C++
UTryllWorkflowAsset* UTryllAgentComponent::WorkflowAsset;
```



Designer-authored workflow asset. Required for agent creation. 


        

<hr>



### variable Workload 

```C++
ETryllWorkload UTryllAgentComponent::Workload;
```



Whether a human ever waits on this agent's turns. Set Background for world-logic/evaluator agents so the inference scheduler paces or defers their work first under GPU pressure. Purely a scheduling hint. 


        

<hr>



### variable bAutoCreateOnConnect 

```C++
bool UTryllAgentComponent::bAutoCreateOnConnect;
```



Automatically create the agent once the session is ready (connected + configured). 


        

<hr>



### variable bEnableDiagnostics 

```C++
bool UTryllAgentComponent::bEnableDiagnostics;
```



Request the server to populate TurnComplete.debug\_info with workflow execution data. Authoring UI retired (Package 2): Chat and Agent Log OR this on via transient override / diagnostics-demand lease. Kept for runtime/Blueprint API and asset compatibility; default remains false (zero overhead in production). 


        

<hr>



### variable bMaintainDialogueHistory 

```C++
bool UTryllAgentComponent::bMaintainDialogueHistory;
```



When true (default) the agent keeps its full dialogue history across turns. When false the history is discarded after each turn — a stateless agent, useful for technical/classification agents that should not store or project prior turns. 


        

<hr>
## Public Functions Documentation




### function AppendInteractions 

```C++
void UTryllAgentComponent::AppendInteractions (
    const TArray< FTryllDialogInteraction > & Interactions
) 
```



Append scripted user/assistant interactions without running the graph. 


        

<hr>



### function Cancel 

```C++
void UTryllAgentComponent::Cancel (
    ETryllCancelMode Mode=ETryllCancelMode::StopAndKeep
) 
```



Cooperatively cancel the in-flight turn. The turn ends with ETryllTurnStatus::Cancelled via OnTurnComplete. No-op if idle or the agent is not ready. Mode selects whether the partial interaction is kept (StopAndKeep, the chat "Stop" button) or discarded (StopAndDiscard). 


        

<hr>



### function ChangeParams 

```C++
void UTryllAgentComponent::ChangeParams (
    const FString & NodeName,
    UTryllNodeParamsBase * Params
) 
```



Mutate a named parameter on a workflow node at runtime. The agent must not be processing a turn. On completion (success or error), OnParamChanged fires.


Use the clone-set-send idiom: auto\* P = Cast&lt;UTryllGenerateParams&gt;(GetNodeParamsBaseline("gen")); auto\* Mut = Cast&lt;UTryllGenerateParams&gt;(UTryllNodeParamsFactory::CloneParams(P, this)); Mut-&gt;SystemPrompt = TEXT("New prompt"); ChangeParams(TEXT("gen"), Mut);




**Parameters:**


* `NodeName` Instance name of the target node in the graph. 
* `Params` Fully specified typed params for the node (clone of baseline + edits). 




        

<hr>



### function CreateAgent 

```C++
void UTryllAgentComponent::CreateAgent () 
```



Request creation of a server-side agent from WorkflowAsset. Fires OnAgentReady on success; OnError on failure (including when no WorkflowAsset is assigned). Safe to call from Blueprint Begin Play. 


        

<hr>



### function DestroyAgent 

```C++
void UTryllAgentComponent::DestroyAgent () 
```



Request destruction of the current server-side agent. OnAgentDestroyed (on the subsystem) fires on completion. 


        

<hr>



### function EvictKvCache 

```C++
void UTryllAgentComponent::EvictKvCache () 
```



Release every eligible language-model backend KV context. 


        

<hr>



### function ExportDialog 

```C++
void UTryllAgentComponent::ExportDialog () 
```



Export the completed dialog. C++ only — snapshot is not Blueprint-exposed. 


        

<hr>



### function FindToolHandler 

```C++
const FTryllToolHandler * UTryllAgentComponent::FindToolHandler (
    const FString & ToolName
) const
```



Look up a registered tool handler without going through an agent, or nullptr when this component has none for that name. Lets an external driver that runs its own agent — the Dialog Lab — invoke the same game-side handler this component would have used, instead of simulating it with authored data.


A handler that keeps per-conversation state must be told when the dialogue restarts, and such a driver's agent is not this component's, so none of the delegates here fire for it. The convention is a parameterless `UFUNCTION()` named `ResetToolState` on the handler's component, which the driver dispatches by name. In-game and Chat-panel callers need nothing extra: the handler binds OnAgentReady and OnInteractionsRemoved itself.


The returned pointer is owned by this component's handler map and is invalidated by RegisterTool / UnregisterTool. Call it, do not store it. 


        

<hr>



### function GetAgent 

```C++
inline const TSharedPtr< FTryllAgent > & UTryllAgentComponent::GetAgent () const
```



Native access to the underlying agent, for C++ callers that need per-call callbacks the Blueprint event surface cannot express — a KV-residency policy managing several agents, for instance, needs to know which prefill finished. Not a UFUNCTION: TSharedPtr is not a Blueprint type. Mirrors the public `Agent` property on the Unity component.


May be null before creation completes; check [**IsAgentReady()**](class_u_tryll_agent_component.md#function-isagentready) first. 


        

<hr>



### function GetAgentId 

```C++
inline int64 UTryllAgentComponent::GetAgentId () const
```



Server-assigned agent id, or 0 if the agent has not been created yet. 


        

<hr>



### function GetKvCacheStatus 

```C++
void UTryllAgentComponent::GetKvCacheStatus () 
```



Query aggregate language-model KV-cache lifecycle state. 


        

<hr>



### function GetLiveVariables 

```C++
FTryllAgentVariables * UTryllAgentComponent::GetLiveVariables () 
```



C++ access to the live variables mirror: the PIE agent when ready, otherwise an editor-preview agent attached by Tryll Chat (if any). Null when neither is active. Used by Details customization for live override apply / Flush Now. 


        

<hr>



### function GetNodeParamsBaseline 

```C++
UTryllNodeParamsBase * UTryllAgentComponent::GetNodeParamsBaseline (
    const FString & NodeName
) const
```



Return the authored params baseline for the named node. Call after OnAgentReady has fired; returns null if the node name is unknown. 


        

<hr>



### function GetPausedNode 

```C++
FString UTryllAgentComponent::GetPausedNode () const
```



Name of the node this agent's turn is paused after, or an empty string when it is not paused. Non-empty means the server is holding the turn open until Resume (or Cancel) — Send Message will keep failing with AgentBusy until one of those arrives. Covers Pause nodes as well as pausing tool calls, unlike Has Paused Tool Batch. 


        

<hr>



### function GetPendingMessage 

```C++
inline const FString & UTryllAgentComponent::GetPendingMessage () const
```



The queued message, or empty. For a "waiting to send" indicator. 


        

<hr>



### function GetVariables 

```C++
UTryllAgentVariables * UTryllAgentComponent::GetVariables () 
```



Blueprint/C++ access to this agent's declared per-agent Variables, or nullptr if no agent is ready. Use to Set/Get/AddToSet/RemoveFromSet/Reset declared variables from Blueprint or C++ — see [**UTryllAgentVariables**](class_u_tryll_agent_variables.md). (C++ code that needs the raw mirror can use the agent's own [**FTryllAgent::GetVariables()**](class_f_tryll_agent.md#function-getvariables).) 


        

<hr>



### function HasLiveAgent 

```C++
bool UTryllAgentComponent::HasLiveAgent () const
```



True when a PIE agent or editor-preview agent is available for live variable edits. 


        

<hr>



### function HasPausedToolBatch 

```C++
bool UTryllAgentComponent::HasPausedToolBatch () const
```



True while a paused tool-call batch is retained: surfaced by the current pause and not yet cleared by a successful resume Ack, turn end/cancel, destroy, or a terminal disconnect. False when no agent is ready. 


        

<hr>



### function HasPendingMessage 

```C++
inline bool UTryllAgentComponent::HasPendingMessage () const
```



True while a message is waiting for the agent to become available. 


        

<hr>



### function ImportDialog 

```C++
void UTryllAgentComponent::ImportDialog (
    const FTryllDialogSnapshot & Snapshot,
    ETryllDialogCompatibility Compatibility=ETryllDialogCompatibility::Strict
) 
```



Replace the agent's dialog. C++ only — snapshot is not Blueprint-exposed. 


        

<hr>



### function IsAgentReady 

```C++
inline bool UTryllAgentComponent::IsAgentReady () const
```



True after the server has confirmed agent creation (OnAgentReady has fired). 


        

<hr>



### function PrefillKvCache 

```C++
void UTryllAgentComponent::PrefillKvCache () 
```



Synchronise every eligible language-model context to its reusable prefix. 


        

<hr>



### function RegisterTool [1/2]

```C++
void UTryllAgentComponent::RegisterTool (
    const FString & ToolName,
    FTryllToolHandler Handler
) 
```



Register native game-thread auto-resume sugar for a named tool. 


        

<hr>



### function RegisterTool [2/2]

```C++
void UTryllAgentComponent::RegisterTool (
    const FString & ToolName,
    FTryllSimpleToolHandler Handler
) 
```




<hr>



### function RemoveInteractionsFromEnd 

```C++
void UTryllAgentComponent::RemoveInteractionsFromEnd (
    int32 Count
) 
```



Remove whole interactions from the end of the dialog. 


        

<hr>



### function Resume 

```C++
void UTryllAgentComponent::Resume (
    const FString & ResumeNode=TEXT("")
) 
```



Resume a turn that is currently paused (see the subsystem's OnPaused event, fired for the Pause node / pausing ToolCall dispositions). On completion (success or error), OnResumed fires.




**Parameters:**


* `ResumeNode` Empty continues via the paused node's pending exit route; non-empty jumps to that node by name. 




        

<hr>



### function ResumeWithToolResult 

```C++
void UTryllAgentComponent::ResumeWithToolResult (
    const FTryllToolResult & ToolResult,
    const FString & ResumeNode=TEXT("")
) 
```



Resume with one result correlated to a tool-call ID. 


        

<hr>



### function ResumeWithToolResults 

```C++
void UTryllAgentComponent::ResumeWithToolResults (
    const TArray< FTryllToolResult > & ToolResults,
    const FString & ResumeNode=TEXT("")
) 
```



Resume with the complete result batch for one paused tool-call pass. 


        

<hr>



### function RetryPausedToolBatch 

```C++
void UTryllAgentComponent::RetryPausedToolBatch () 
```



Re-attempt the retained paused tool-call batch: handlers that already produced a result on a previous attempt are skipped, remaining handlers run, and a fresh Resume is sent. No-op when no agent is ready or there is no retained batch. 


        

<hr>



### function SendMessage 

```C++
void UTryllAgentComponent::SendMessage (
    const FString & Message
) 
```



Send a user-turn message to the agent. Streaming tokens arrive on OnAnswerText; the turn ends with OnTurnComplete + OnAnswerFull. No-op if the agent is not yet ready ([**IsAgentReady()**](class_u_tryll_agent_component.md#function-isagentready) == false).


The message is **queued, not dropped**, when the agent exists but cannot accept it yet. An agent refuses a send with AgentBusy while a turn is running or a KV-cache operation is in flight — and a cache operation is exactly what happens when a game prefills the agent the player has just selected. Without queueing, a message typed during that window is silently lost and the player must retype it, which is the worst available outcome. The queue is flushed automatically when the agent becomes ready, when a turn completes, and when a cache operation finishes.


Only the most recent message is held: if two are sent before either reaches the agent, the newer line is the one the player meant. OnMessageQueued fires when a send is deferred. 


        

<hr>



### function TryFlushPendingMessage 

```C++
bool UTryllAgentComponent::TryFlushPendingMessage () 
```



Hands the queued message to the agent if it can accept one now. Safe and cheap to call repeatedly; returns true only when a message was actually handed over.


Called automatically on agent-ready, turn-complete and cache-operation-complete. It is public because a KV-residency policy that drives prefills through [**FTryllAgent**](class_f_tryll_agent.md) directly — rather than through this component — closes a busy window this component never observed, and should nudge the queue when it does. 


        

<hr>



### function UTryllAgentComponent 

```C++
UTryllAgentComponent::UTryllAgentComponent () 
```




<hr>



### function UnregisterTool 

```C++
bool UTryllAgentComponent::UnregisterTool (
    const FString & ToolName
) 
```



Remove a previously registered native tool handler. 


        

<hr>
## Protected Functions Documentation




### function BeginPlay 

```C++
virtual void UTryllAgentComponent::BeginPlay () override
```




<hr>



### function EndPlay 

```C++
virtual void UTryllAgentComponent::EndPlay (
    const EEndPlayReason::Type EndPlayReason
) override
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllAgentComponent.h`

