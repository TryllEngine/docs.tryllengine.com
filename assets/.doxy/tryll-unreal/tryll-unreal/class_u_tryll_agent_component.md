

# Class UTryllAgentComponent



[**ClassList**](annotated.md) **>** [**UTryllAgentComponent**](class_u_tryll_agent_component.md)



[More...](#detailed-description)

* `#include <TryllAgentComponent.h>`



Inherits the following classes: UActorComponent


















## Public Attributes

| Type | Name |
| ---: | :--- |
|  [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) | [**InlineGraphDescription**](#variable-inlinegraphdescription)  <br> |
|  TArray&lt; [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) &gt; | [**InlineVariables**](#variable-inlinevariables)  <br> |
|  ETryllKvCacheInitialization | [**KvCacheInitialization**](#variable-kvcacheinitialization)   = `ETryllKvCacheInitialization::AllocateOnly`<br> |
|  FOnTryllAgentReady | [**OnAgentReady**](#variable-onagentready)  <br> |
|  FOnTryllAgentAnswerFull | [**OnAnswerFull**](#variable-onanswerfull)  <br> |
|  FOnTryllAgentAnswerText | [**OnAnswerText**](#variable-onanswertext)  <br> |
|  FOnTryllAppendInteractionsError | [**OnAppendInteractionsError**](#variable-onappendinteractionserror)  <br> |
|  FOnTryllAgentError | [**OnError**](#variable-onerror)  <br> |
|  FOnTryllInteractionsAppended | [**OnInteractionsAppended**](#variable-oninteractionsappended)  <br> |
|  FOnTryllInteractionsRemoved | [**OnInteractionsRemoved**](#variable-oninteractionsremoved)  <br> |
|  FOnTryllKvCacheEvictError | [**OnKvCacheEvictError**](#variable-onkvcacheevicterror)  <br> |
|  FOnTryllKvCacheEvicted | [**OnKvCacheEvicted**](#variable-onkvcacheevicted)  <br> |
|  FOnTryllKvCachePrefillError | [**OnKvCachePrefillError**](#variable-onkvcacheprefillerror)  <br> |
|  FOnTryllKvCachePrefilled | [**OnKvCachePrefilled**](#variable-onkvcacheprefilled)  <br> |
|  FOnTryllKvCacheStatus | [**OnKvCacheStatus**](#variable-onkvcachestatus)  <br> |
|  FOnTryllKvCacheStatusError | [**OnKvCacheStatusError**](#variable-onkvcachestatuserror)  <br> |
|  FOnTryllParamChanged | [**OnParamChanged**](#variable-onparamchanged)  <br> |
|  FOnTryllRemoveInteractionsError | [**OnRemoveInteractionsError**](#variable-onremoveinteractionserror)  <br> |
|  FOnTryllResumed | [**OnResumed**](#variable-onresumed)  <br> |
|  FOnTryllAgentTurnComplete | [**OnTurnComplete**](#variable-onturncomplete)  <br> |
|  TObjectPtr&lt; [**UTryllSpeakerComponent**](class_u_tryll_speaker_component.md) &gt; | [**Speaker**](#variable-speaker)   = `nullptr`<br> |
|  TArray&lt; [**FTryllVariableOverride**](struct_f_tryll_variable_override.md) &gt; | [**VariableOverrides**](#variable-variableoverrides)  <br> |
|  [**UTryllWorkflowAsset**](class_u_tryll_workflow_asset.md) \* | [**WorkflowAsset**](#variable-workflowasset)   = `nullptr`<br> |
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
|  int64 | [**GetAgentId**](#function-getagentid) () const<br> |
|  void | [**GetKvCacheStatus**](#function-getkvcachestatus) () <br> |
|  [**FTryllAgentVariables**](class_f_tryll_agent_variables.md) \* | [**GetLiveVariables**](#function-getlivevariables) () <br> |
|  [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* | [**GetNodeParamsBaseline**](#function-getnodeparamsbaseline) (const FString & NodeName) const<br> |
|  TArray&lt; FString &gt; | [**GetStartNodeOptions**](#function-getstartnodeoptions) () const<br> |
|  [**UTryllAgentVariables**](class_u_tryll_agent_variables.md) \* | [**GetVariables**](#function-getvariables) () <br> |
|  bool | [**HasLiveAgent**](#function-hasliveagent) () const<br> |
|  bool | [**HasPausedToolBatch**](#function-haspausedtoolbatch) () const<br> |
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
|   | [**UTryllAgentComponent**](#function-utryllagentcomponent) () <br> |
|  bool | [**UnregisterTool**](#function-unregistertool) (const FString & ToolName) <br> |
























## Protected Functions

| Type | Name |
| ---: | :--- |
| virtual void | [**BeginPlay**](#function-beginplay) () override<br> |
| virtual void | [**EndPlay**](#function-endplay) (const EEndPlayReason::Type EndPlayReason) override<br> |




## Detailed Description


Actor component: manages one Tryll server-side agent. Attach to any Actor; Blueprint users bind to the delegates and call [**SendMessage()**](class_u_tryll_agent_component.md#function-sendmessage).


Graph resolution order:
* WorkflowAsset (if set) — designer-authored data asset in the Content Browser.
* InlineGraphDescription — set programmatically in C++ or edited in the Details panel.




The component auto-creates an agent once the session is ready (if bAutoCreateOnConnect) — it queues until OnCreateSessionComplete if it spawns before the session is configured. It auto-destroys the agent on EndPlay. 


    
## Public Attributes Documentation




### variable InlineGraphDescription 

```C++
FTryllGraphDescription UTryllAgentComponent::InlineGraphDescription;
```



Fallback graph description used when WorkflowAsset is not set. Can be populated programmatically via FTryllGraphBuilder::Build(). 


        

<hr>



### variable InlineVariables 

```C++
TArray<FTryllVariableDecl> UTryllAgentComponent::InlineVariables;
```



Declared per-agent Variables when WorkflowAsset is not set (WorkflowAsset-&gt;Variables is used instead when one is). 


        

<hr>



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



Designer-authored workflow asset. When set, its Graph takes precedence over InlineGraphDescription for agent creation. 


        

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



Request the server to populate TurnComplete.debug\_info with workflow execution data. Intended for development and QA; off by default for zero overhead in production. 


        

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



Request creation of a server-side agent from WorkflowAsset (if set) or InlineGraphDescription. Fires OnAgentReady on success; OnError on failure. Safe to call from Blueprint Begin Play. 


        

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



### function GetStartNodeOptions 

```C++
inline TArray< FString > UTryllAgentComponent::GetStartNodeOptions () const
```



Options provider for FTryllGraphDescription::StartNode dropdown in the Details panel. 


        

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
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllAgentComponent.h`

