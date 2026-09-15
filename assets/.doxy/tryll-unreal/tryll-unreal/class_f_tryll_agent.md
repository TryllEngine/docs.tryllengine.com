

# Class FTryllAgent



[**ClassList**](annotated.md) **>** [**FTryllAgent**](class_f_tryll_agent.md)



[More...](#detailed-description)

* `#include <TryllAgent.h>`



Inherits the following classes: TSharedFromThis< FTryllAgent >,  FGCObject


































## Public Functions

| Type | Name |
| ---: | :--- |
| virtual void | [**AddReferencedObjects**](#function-addreferencedobjects) (FReferenceCollector & Collector) override<br> |
|  void | [**AppendInteractions**](#function-appendinteractions) (const TArray&lt; [**FTryllDialogInteraction**](struct_f_tryll_dialog_interaction.md) &gt; & Interactions, TFunction&lt; void(uint32 Count, const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete=nullptr) <br> |
|  void | [**Cancel**](#function-cancel) (ETryllCancelMode Mode=ETryllCancelMode::StopAndKeep) <br> |
|  void | [**ChangeParams**](#function-changeparams) (const FString & NodeName, [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* Params, TFunction&lt; void(const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete=nullptr) <br> |
|  void | [**ClearAfterDestroyRequest**](#function-clearafterdestroyrequest) () <br> |
|  void | [**ClearPausedToolBatch**](#function-clearpausedtoolbatch) () <br> |
|  void | [**DispatchError**](#function-dispatcherror) (const [**FTryllError**](struct_f_tryll_error.md) & Error) const<br> |
|  void | [**DrainPendingLifecycleCallbacks**](#function-drainpendinglifecyclecallbacks) (const [**FTryllError**](struct_f_tryll_error.md) & TerminalError) <br> |
|  void | [**EvictKvCache**](#function-evictkvcache) (TFunction&lt; void(const [**FTryllKvCacheEvictResult**](struct_f_tryll_kv_cache_evict_result.md) &, const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete=nullptr) <br> |
|  void | [**ExportDialog**](#function-exportdialog) (TFunction&lt; void(const [**FTryllDialogSnapshot**](struct_f_tryll_dialog_snapshot.md) &, const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete=nullptr) <br> |
|   | [**FTryllAgent**](#function-ftryllagent) (TSharedPtr&lt; FTryllConnection &gt; InConnection, std::uint64\_t InAgentId, const TArray&lt; [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) &gt; & InDeclaredVariables={}) <br> |
|  std::uint64\_t | [**GetAgentId**](#function-getagentid) () noexcept const<br> |
|  const FString & | [**GetDisplayName**](#function-getdisplayname) () const<br> |
|  void | [**GetKvCacheStatus**](#function-getkvcachestatus) (TFunction&lt; void(const [**FTryllKvCacheStatus**](struct_f_tryll_kv_cache_status.md) &, const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete=nullptr) <br> |
|  [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* | [**GetNodeParamsBaseline**](#function-getnodeparamsbaseline) (const FString & NodeName) const<br> |
|  const FString & | [**GetPausedNode**](#function-getpausednode) () const<br> |
| virtual FString | [**GetReferencerName**](#function-getreferencername) () override const<br> |
|  [**FTryllAgentVariables**](class_f_tryll_agent_variables.md) & | [**GetVariables**](#function-getvariables) () const<br> |
|  void | [**HandleAnswerText**](#function-handleanswertext) (const FString & NodeName, const FString & Text, bool bIsDelta, bool bIsFinal) <br> |
|  void | [**HandleAppendInteractionsResult**](#function-handleappendinteractionsresult) (std::uint64\_t RequestId, uint32 Count, const [**FTryllError**](struct_f_tryll_error.md) & Error) <br> |
|  void | [**HandleChangeParamResult**](#function-handlechangeparamresult) (std::uint64\_t RequestId, const [**FTryllError**](struct_f_tryll_error.md) & Error) <br> |
|  void | [**HandleError**](#function-handleerror) (const [**FTryllError**](struct_f_tryll_error.md) & Error) <br> |
|  void | [**HandleEvictKvCacheResult**](#function-handleevictkvcacheresult) (std::uint64\_t RequestId, const [**FTryllKvCacheEvictResult**](struct_f_tryll_kv_cache_evict_result.md) & Result, const [**FTryllError**](struct_f_tryll_error.md) & Error) <br> |
|  void | [**HandleExportDialogResult**](#function-handleexportdialogresult) (std::uint64\_t RequestId, const [**FTryllDialogSnapshot**](struct_f_tryll_dialog_snapshot.md) & Snapshot, const [**FTryllError**](struct_f_tryll_error.md) & Error) <br> |
|  void | [**HandleGetVariablesResult**](#function-handlegetvariablesresult) (std::uint64\_t RequestId, const TArray&lt; [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) &gt; & Variables, const [**FTryllError**](struct_f_tryll_error.md) & Error) <br> |
|  void | [**HandleImportDialogResult**](#function-handleimportdialogresult) (std::uint64\_t RequestId, const [**FTryllDialogImportResult**](struct_f_tryll_dialog_import_result.md) & Result, const [**FTryllError**](struct_f_tryll_error.md) & Error) <br> |
|  void | [**HandleKvCacheStatusResult**](#function-handlekvcachestatusresult) (std::uint64\_t RequestId, const [**FTryllKvCacheStatus**](struct_f_tryll_kv_cache_status.md) & Result, const [**FTryllError**](struct_f_tryll_error.md) & Error) <br> |
|  void | [**HandlePaused**](#function-handlepaused) (const FString & NodeName, bool bExternalListenerBound) <br> |
|  void | [**HandlePrefillKvCacheResult**](#function-handleprefillkvcacheresult) (std::uint64\_t RequestId, const [**FTryllKvCachePrefillResult**](struct_f_tryll_kv_cache_prefill_result.md) & Result, const [**FTryllError**](struct_f_tryll_error.md) & Error) <br> |
|  void | [**HandleRemoveInteractionsFromEndResult**](#function-handleremoveinteractionsfromendresult) (std::uint64\_t RequestId, uint32 Count, const [**FTryllError**](struct_f_tryll_error.md) & Error) <br> |
|  void | [**HandleResumeResult**](#function-handleresumeresult) (std::uint64\_t RequestId, const [**FTryllError**](struct_f_tryll_error.md) & Error) <br> |
|  void | [**HandleToolCall**](#function-handletoolcall) (const FString & CallId, const FString & NodeName, const FString & ToolName, const FString & ArgumentsJson, const FString & Disposition={}) <br> |
|  void | [**HandleTtsAudio**](#function-handlettsaudio) (const FString & NodeName, TArrayView&lt; const int16 &gt; Pcm) <br> |
|  void | [**HandleTtsAudioFormat**](#function-handlettsaudioformat) (const FString & NodeName, uint32 SampleRate, uint16 Channels, uint16 BitsPerSample) <br> |
|  void | [**HandleTurnComplete**](#function-handleturncomplete) (ETryllTurnStatus Status, const FString & DebugInfo, int32 TokensGenerated) <br> |
|  void | [**HandleUpdateVariablesResult**](#function-handleupdatevariablesresult) (std::uint64\_t RequestId, const [**FTryllError**](struct_f_tryll_error.md) & Error) <br> |
|  bool | [**HasPausedToolBatch**](#function-haspausedtoolbatch) () const<br> |
|  void | [**ImportDialog**](#function-importdialog) (const [**FTryllDialogSnapshot**](struct_f_tryll_dialog_snapshot.md) & Snapshot, ETryllDialogCompatibility Compatibility=ETryllDialogCompatibility::Strict, TFunction&lt; void(const [**FTryllDialogImportResult**](struct_f_tryll_dialog_import_result.md) &, const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete=nullptr) <br> |
|  bool | [**IsValid**](#function-isvalid) () noexcept const<br> |
|  void | [**PrefillKvCache**](#function-prefillkvcache) (TFunction&lt; void(const [**FTryllKvCachePrefillResult**](struct_f_tryll_kv_cache_prefill_result.md) &, const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete=nullptr) <br> |
|  void | [**RegisterTool**](#function-registertool-12) (const FString & ToolName, FTryllToolHandler Handler) <br> |
|  void | [**RegisterTool**](#function-registertool-22) (const FString & ToolName, FTryllSimpleToolHandler Handler) <br> |
|  void | [**RemoveInteractionsFromEnd**](#function-removeinteractionsfromend) (int32 Count, TFunction&lt; void(uint32 RemovedCount, const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete=nullptr) <br> |
|  void | [**Resume**](#function-resume) (const FString & ResumeNode=FString{}, TFunction&lt; void(const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete=nullptr) <br> |
|  void | [**ResumeWithToolResult**](#function-resumewithtoolresult) (const [**FTryllToolResult**](struct_f_tryll_tool_result.md) & ToolResult, const FString & ResumeNode=FString{}, TFunction&lt; void(const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete=nullptr) <br> |
|  void | [**ResumeWithToolResults**](#function-resumewithtoolresults) (const TArray&lt; [**FTryllToolResult**](struct_f_tryll_tool_result.md) &gt; & ToolResults, const FString & ResumeNode=FString{}, TFunction&lt; void(const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete=nullptr) <br> |
|  void | [**RetryPausedToolBatch**](#function-retrypausedtoolbatch) () <br> |
|  void | [**SendGetVariables**](#function-sendgetvariables) (TArray&lt; FString &gt; Names, TFunction&lt; void(const TArray&lt; [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) &gt; &, const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete) <br> |
|  void | [**SendMessage**](#function-sendmessage) (const FString & Text) <br> |
|  void | [**SendUpdateVariables**](#function-sendupdatevariables) (TArray&lt; [**FTryllVariableUpdate**](struct_f_tryll_variable_update.md) &gt; Updates, TFunction&lt; void(const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete, uint64 CarrierRequestId=0) <br> |
|  void | [**SetCreatedWithDiagnostics**](#function-setcreatedwithdiagnostics) (bool bValue) <br> |
|  void | [**SetDisplayName**](#function-setdisplayname) (FString InName) <br> |
|  void | [**SetNodeParamsBaseline**](#function-setnodeparamsbaseline) (const FString & NodeName, [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* Params) <br> |
|  void | [**SetOnAnswerText**](#function-setonanswertext) (TFunction&lt; void(const FString &NodeName, const FString &Text, bool bIsDelta, bool bIsFinal)&gt; Callback) <br> |
|  void | [**SetOnError**](#function-setonerror) (TFunction&lt; void(const [**FTryllError**](struct_f_tryll_error.md) &Error)&gt; Callback) <br> |
|  void | [**SetOnToolHandlerError**](#function-setontoolhandlererror) (TFunction&lt; void(const [**FTryllError**](struct_f_tryll_error.md) &)&gt; Callback) <br> |
|  void | [**SetOnTtsAudio**](#function-setonttsaudio) (TFunction&lt; void(const FString &NodeName, TArrayView&lt; const int16 &gt; Pcm)&gt; Callback) <br> |
|  void | [**SetOnTtsAudioFormat**](#function-setonttsaudioformat) (TFunction&lt; void(const FString &NodeName, uint32 SampleRate, uint16 Channels, uint16 BitsPerSample)&gt; Callback) <br> |
|  void | [**SetOnTurnComplete**](#function-setonturncomplete) (TFunction&lt; void(ETryllTurnStatus Status, const FString &DebugInfo, int32 TokensGenerated)&gt; Callback) <br> |
|  bool | [**UnregisterTool**](#function-unregistertool) (const FString & ToolName) <br> |
|  bool | [**WasCreatedWithDiagnostics**](#function-wascreatedwithdiagnostics) () noexcept const<br> |
|   | [**~FTryllAgent**](#function-ftryllagent) () <br> |




























## Detailed Description


Plain C++ handle for a server-side agent. Created by [**UTryllSubsystem**](class_u_tryll_subsystem.md) after a successful CreateAgentResponse.


Ownership: typically held as TSharedPtr&lt;FTryllAgent&gt; by [**UTryllAgentComponent**](class_u_tryll_agent_component.md). Lifetime: deterministic — the owning component releases the shared\_ptr; this triggers PushDestroyAgent via [**UTryllSubsystem::RequestDestroyAgent()**](class_u_tryll_subsystem.md#function-requestdestroyagent).


All callbacks are invoked on the game thread (from UTryllSubsystem::Tick). 


    
## Public Functions Documentation




### function AddReferencedObjects 

```C++
virtual void FTryllAgent::AddReferencedObjects (
    FReferenceCollector & Collector
) override
```




<hr>



### function AppendInteractions 

```C++
void FTryllAgent::AppendInteractions (
    const TArray< FTryllDialogInteraction > & Interactions,
    TFunction< void(uint32 Count, const FTryllError &)> OnComplete=nullptr
) 
```



Append scripted user/assistant interactions without running the graph. Idle-only. 


        

<hr>



### function Cancel 

```C++
void FTryllAgent::Cancel (
    ETryllCancelMode Mode=ETryllCancelMode::StopAndKeep
) 
```



Cooperatively cancel the agent's in-flight turn. The turn ends with ETryllTurnStatus::Cancelled via the OnTurnComplete callback. No-op if idle. Mode selects whether the partial interaction is kept (default) or discarded. 


        

<hr>



### function ChangeParams 

```C++
void FTryllAgent::ChangeParams (
    const FString & NodeName,
    UTryllNodeParamsBase * Params,
    TFunction< void(const FTryllError &)> OnComplete=nullptr
) 
```



Apply typed node parameters to a workflow node at runtime. The agent must not be processing a turn. Completion is invoked on the game thread via the OnComplete callback.


Clone-set-send idiom (C++): auto\* P = [**UTryllNodeParamsFactory::CloneParams**](class_u_tryll_node_params_factory.md#function-cloneparams)(AgentComp-&gt;GetNodeParamsBaseline("gen"), this); Cast&lt;UTryllGenerateParams&gt;(P)-&gt;SystemPrompt = TEXT("New prompt"); Agent-&gt;ChangeParams(TEXT("gen"), P); 


        

<hr>



### function ClearAfterDestroyRequest 

```C++
void FTryllAgent::ClearAfterDestroyRequest () 
```



Called by [**UTryllSubsystem**](class_u_tryll_subsystem.md) when the destroy Ack arrives; clears the local agent id. 


        

<hr>



### function ClearPausedToolBatch 

```C++
void FTryllAgent::ClearPausedToolBatch () 
```



Discard the retained paused tool-call batch without resuming (terminal disconnect). 


        

<hr>



### function DispatchError 

```C++
void FTryllAgent::DispatchError (
    const FTryllError & Error
) const
```



Invoke the OnError callback (if set). Used by [**FTryllAgentVariables**](class_f_tryll_agent_variables.md) to surface an auto-flush rejection on the agent's normal error path. 


        

<hr>



### function DrainPendingLifecycleCallbacks 

```C++
void FTryllAgent::DrainPendingLifecycleCallbacks (
    const FTryllError & TerminalError
) 
```



Fail in-flight KV-cache lifecycle callbacks before the agent handle is cleared. 


        

<hr>



### function EvictKvCache 

```C++
void FTryllAgent::EvictKvCache (
    TFunction< void(const FTryllKvCacheEvictResult &, const FTryllError &)> OnComplete=nullptr
) 
```



Release backend KV contexts for every eligible language-model node. 


        

<hr>



### function ExportDialog 

```C++
void FTryllAgent::ExportDialog (
    TFunction< void(const FTryllDialogSnapshot &, const FTryllError &)> OnComplete=nullptr
) 
```



Export the completed dialog as an owned snapshot. Idle-only. Does not write a file. 


        

<hr>



### function FTryllAgent 

```C++
FTryllAgent::FTryllAgent (
    TSharedPtr< FTryllConnection > InConnection,
    std::uint64_t InAgentId,
    const TArray< FTryllVariableDecl > & InDeclaredVariables={}
) 
```




<hr>



### function GetAgentId 

```C++
inline std::uint64_t FTryllAgent::GetAgentId () noexcept const
```




<hr>



### function GetDisplayName 

```C++
inline const FString & FTryllAgent::GetDisplayName () const
```




<hr>



### function GetKvCacheStatus 

```C++
void FTryllAgent::GetKvCacheStatus (
    TFunction< void(const FTryllKvCacheStatus &, const FTryllError &)> OnComplete=nullptr
) 
```



Query aggregate language-model KV-cache lifecycle state. 


        

<hr>



### function GetNodeParamsBaseline 

```C++
UTryllNodeParamsBase * FTryllAgent::GetNodeParamsBaseline (
    const FString & NodeName
) const
```



Return a deep copy of the last authored params for the named node. Use this as the starting point for a clone-set-send mutation; structural fields are automatically propagated from the baseline. 


        

<hr>



### function GetPausedNode 

```C++
inline const FString & FTryllAgent::GetPausedNode () const
```



Name of the node this agent's turn is currently paused after, or an empty string when the turn is not paused. A non-empty value means the server is holding the turn open until Resume (or Cancel) — new messages are rejected with AgentBusy until then. 


        

<hr>



### function GetReferencerName 

```C++
inline virtual FString FTryllAgent::GetReferencerName () override const
```




<hr>



### function GetVariables 

```C++
inline FTryllAgentVariables & FTryllAgent::GetVariables () const
```



Local mirror of this agent's declared per-agent Variables. 


        

<hr>



### function HandleAnswerText 

```C++
void FTryllAgent::HandleAnswerText (
    const FString & NodeName,
    const FString & Text,
    bool bIsDelta,
    bool bIsFinal
) 
```




<hr>



### function HandleAppendInteractionsResult 

```C++
void FTryllAgent::HandleAppendInteractionsResult (
    std::uint64_t RequestId,
    uint32 Count,
    const FTryllError & Error
) 
```




<hr>



### function HandleChangeParamResult 

```C++
void FTryllAgent::HandleChangeParamResult (
    std::uint64_t RequestId,
    const FTryllError & Error
) 
```



Invoked by [**UTryllSubsystem**](class_u_tryll_subsystem.md) when a ChangeParamOk or ChangeParamErr arrives. 


        

<hr>



### function HandleError 

```C++
void FTryllAgent::HandleError (
    const FTryllError & Error
) 
```




<hr>



### function HandleEvictKvCacheResult 

```C++
void FTryllAgent::HandleEvictKvCacheResult (
    std::uint64_t RequestId,
    const FTryllKvCacheEvictResult & Result,
    const FTryllError & Error
) 
```




<hr>



### function HandleExportDialogResult 

```C++
void FTryllAgent::HandleExportDialogResult (
    std::uint64_t RequestId,
    const FTryllDialogSnapshot & Snapshot,
    const FTryllError & Error
) 
```




<hr>



### function HandleGetVariablesResult 

```C++
void FTryllAgent::HandleGetVariablesResult (
    std::uint64_t RequestId,
    const TArray< FTryllVariableDecl > & Variables,
    const FTryllError & Error
) 
```



Invoked by [**UTryllSubsystem**](class_u_tryll_subsystem.md) when a GetVariablesOk or GetVariablesErr arrives. 


        

<hr>



### function HandleImportDialogResult 

```C++
void FTryllAgent::HandleImportDialogResult (
    std::uint64_t RequestId,
    const FTryllDialogImportResult & Result,
    const FTryllError & Error
) 
```




<hr>



### function HandleKvCacheStatusResult 

```C++
void FTryllAgent::HandleKvCacheStatusResult (
    std::uint64_t RequestId,
    const FTryllKvCacheStatus & Result,
    const FTryllError & Error
) 
```




<hr>



### function HandlePaused 

```C++
void FTryllAgent::HandlePaused (
    const FString & NodeName,
    bool bExternalListenerBound
) 
```





**Parameters:**


* `NodeName` node the turn is parked after. 
* `bExternalListenerBound` true when a client delegate (the subsystem's OnPaused) is listening, so a pause that this agent cannot auto-resume may still be resumed by game code. When false and no registered tool handler covers the batch, nothing can resume the turn and a warning is logged. 




        

<hr>



### function HandlePrefillKvCacheResult 

```C++
void FTryllAgent::HandlePrefillKvCacheResult (
    std::uint64_t RequestId,
    const FTryllKvCachePrefillResult & Result,
    const FTryllError & Error
) 
```



Invoked by [**UTryllSubsystem**](class_u_tryll_subsystem.md) when a KV-cache lifecycle response arrives. 


        

<hr>



### function HandleRemoveInteractionsFromEndResult 

```C++
void FTryllAgent::HandleRemoveInteractionsFromEndResult (
    std::uint64_t RequestId,
    uint32 Count,
    const FTryllError & Error
) 
```




<hr>



### function HandleResumeResult 

```C++
void FTryllAgent::HandleResumeResult (
    std::uint64_t RequestId,
    const FTryllError & Error
) 
```



Invoked by [**UTryllSubsystem**](class_u_tryll_subsystem.md) when a ResumeOk or ResumeErr arrives. 


        

<hr>



### function HandleToolCall 

```C++
void FTryllAgent::HandleToolCall (
    const FString & CallId,
    const FString & NodeName,
    const FString & ToolName,
    const FString & ArgumentsJson,
    const FString & Disposition={}
) 
```




<hr>



### function HandleTtsAudio 

```C++
void FTryllAgent::HandleTtsAudio (
    const FString & NodeName,
    TArrayView< const int16 > Pcm
) 
```




<hr>



### function HandleTtsAudioFormat 

```C++
void FTryllAgent::HandleTtsAudioFormat (
    const FString & NodeName,
    uint32 SampleRate,
    uint16 Channels,
    uint16 BitsPerSample
) 
```




<hr>



### function HandleTurnComplete 

```C++
void FTryllAgent::HandleTurnComplete (
    ETryllTurnStatus Status,
    const FString & DebugInfo,
    int32 TokensGenerated
) 
```




<hr>



### function HandleUpdateVariablesResult 

```C++
void FTryllAgent::HandleUpdateVariablesResult (
    std::uint64_t RequestId,
    const FTryllError & Error
) 
```



Invoked by [**UTryllSubsystem**](class_u_tryll_subsystem.md) when an UpdateVariablesOk or UpdateVariablesErr arrives. 


        

<hr>



### function HasPausedToolBatch 

```C++
inline bool FTryllAgent::HasPausedToolBatch () const
```



True while a paused tool-call batch is retained: surfaced by the current pause and not yet cleared by a successful resume Ack, turn end/cancel, destroy, or a terminal disconnect. A retained batch may already have some handlers succeeded (see RetryPausedToolBatch). 


        

<hr>



### function ImportDialog 

```C++
void FTryllAgent::ImportDialog (
    const FTryllDialogSnapshot & Snapshot,
    ETryllDialogCompatibility Compatibility=ETryllDialogCompatibility::Strict,
    TFunction< void(const FTryllDialogImportResult &, const FTryllError &)> OnComplete=nullptr
) 
```



Replace the agent's dialog with Snapshot. Idle-only. Never appends. 


        

<hr>



### function IsValid 

```C++
inline bool FTryllAgent::IsValid () noexcept const
```




<hr>



### function PrefillKvCache 

```C++
void FTryllAgent::PrefillKvCache (
    TFunction< void(const FTryllKvCachePrefillResult &, const FTryllError &)> OnComplete=nullptr
) 
```



Synchronise every eligible language-model context to its reusable prefix. 


        

<hr>



### function RegisterTool [1/2]

```C++
void FTryllAgent::RegisterTool (
    const FString & ToolName,
    FTryllToolHandler Handler
) 
```



Register game-thread tool sugar. A paused batch auto-resumes only when every call has a handler and every handler reports success. 


        

<hr>



### function RegisterTool [2/2]

```C++
void FTryllAgent::RegisterTool (
    const FString & ToolName,
    FTryllSimpleToolHandler Handler
) 
```




<hr>



### function RemoveInteractionsFromEnd 

```C++
void FTryllAgent::RemoveInteractionsFromEnd (
    int32 Count,
    TFunction< void(uint32 RemovedCount, const FTryllError &)> OnComplete=nullptr
) 
```



Remove whole interactions from the end of the dialog. count=0 is a no-op. Idle-only. 


        

<hr>



### function Resume 

```C++
void FTryllAgent::Resume (
    const FString & ResumeNode=FString{},
    TFunction< void(const FTryllError &)> OnComplete=nullptr
) 
```



Resume a turn that is currently paused (see the Pause node / typed OnNodeEvent event\_type="paused" wiring on [**UTryllSubsystem**](class_u_tryll_subsystem.md)). ResumeNode empty continues via the paused node's pending exit route; non-empty jumps to that node by name. Completion is invoked on the game thread: nullopt-equivalent (Error.IsOk()) on success, AgentNotPaused (3012) if the agent is not currently paused, or UnknownNode (3005) if ResumeNode does not name a node in the graph. 


        

<hr>



### function ResumeWithToolResult 

```C++
void FTryllAgent::ResumeWithToolResult (
    const FTryllToolResult & ToolResult,
    const FString & ResumeNode=FString{},
    TFunction< void(const FTryllError &)> OnComplete=nullptr
) 
```



Resume with one result correlated to a tool-call ID. 


        

<hr>



### function ResumeWithToolResults 

```C++
void FTryllAgent::ResumeWithToolResults (
    const TArray< FTryllToolResult > & ToolResults,
    const FString & ResumeNode=FString{},
    TFunction< void(const FTryllError &)> OnComplete=nullptr
) 
```



Resume with the complete result batch for one paused tool-call pass. 


        

<hr>



### function RetryPausedToolBatch 

```C++
void FTryllAgent::RetryPausedToolBatch () 
```



Re-attempt the retained paused tool-call batch: handlers that already produced a result on a previous attempt are skipped (not re-run — no duplicate side effects), remaining handlers run, and a fresh Resume is sent. No-op if there is no retained batch ([**HasPausedToolBatch()**](class_f_tryll_agent.md#function-haspausedtoolbatch) is false). Use after OnToolHandlerError reports a failed handler or a rejected/disconnected resume. 


        

<hr>



### function SendGetVariables 

```C++
void FTryllAgent::SendGetVariables (
    TArray< FString > Names,
    TFunction< void(const TArray< FTryllVariableDecl > &, const FTryllError &)> OnComplete
) 
```



Send a GetAgentVariablesRequest. Empty Names = every declared variable. 


        

<hr>



### function SendMessage 

```C++
void FTryllAgent::SendMessage (
    const FString & Text
) 
```



Send a text message to the agent. Response arrives via the OnAnswerText / OnTurnComplete callbacks. 


        

<hr>



### function SendUpdateVariables 

```C++
void FTryllAgent::SendUpdateVariables (
    TArray< FTryllVariableUpdate > Updates,
    TFunction< void(const FTryllError &)> OnComplete,
    uint64 CarrierRequestId=0
) 
```



Send a batched UpdateAgentVariablesRequest (deferred-flush from [**FTryllAgentVariables::FlushIfDirty**](class_f_tryll_agent_variables.md#function-flushifdirty)). 


        

<hr>



### function SetCreatedWithDiagnostics 

```C++
inline void FTryllAgent::SetCreatedWithDiagnostics (
    bool bValue
) 
```




<hr>



### function SetDisplayName 

```C++
inline void FTryllAgent::SetDisplayName (
    FString InName
) 
```



Optional editor/display label (e.g. owning actor name). Not on the wire. 


        

<hr>



### function SetNodeParamsBaseline 

```C++
void FTryllAgent::SetNodeParamsBaseline (
    const FString & NodeName,
    UTryllNodeParamsBase * Params
) 
```



Store/update the authored params baseline for a node (called by the subsystem). 


        

<hr>



### function SetOnAnswerText 

```C++
void FTryllAgent::SetOnAnswerText (
    TFunction< void(const FString &NodeName, const FString &Text, bool bIsDelta, bool bIsFinal)> Callback
) 
```



Streaming text callback. Matches the current AnswerText wire message:
* NodeName = the workflow node that produced this chunk (multi-sender attribution; ignore it if the graph only ever has one text-producing node).
* bIsDelta = true → Text is an incremental token/chunk.
* bIsDelta = false → Text is the full response accumulated so far.
* bIsFinal = true → This is the last AnswerText before TurnComplete for this node. 




        

<hr>



### function SetOnError 

```C++
void FTryllAgent::SetOnError (
    TFunction< void(const FTryllError &Error)> Callback
) 
```



Called on any send-side error (connection loss, server error response, etc.). 


        

<hr>



### function SetOnToolHandlerError 

```C++
void FTryllAgent::SetOnToolHandlerError (
    TFunction< void(const FTryllError &)> Callback
) 
```



Set the callback invoked on the game thread when RegisterTool handler execution or its automatic result-bearing Resume fails. The turn remains paused. Mirrors the Unity client's ToolHandlerError event. 


        

<hr>



### function SetOnTtsAudio 

```C++
void FTryllAgent::SetOnTtsAudio (
    TFunction< void(const FString &NodeName, TArrayView< const int16 > Pcm)> Callback
) 
```



Streaming TTS audio callback. Fires for each TtsAudioFrame within a turn. NodeName is the producing node's name (multi-sender attribution). The view references the FTryllConnEvent storage and is valid only for the duration of the callback. End-of-stream is signalled by OnTurnComplete — TtsAudioFrame intentionally carries no is\_final field. 


        

<hr>



### function SetOnTtsAudioFormat 

```C++
void FTryllAgent::SetOnTtsAudioFormat (
    TFunction< void(const FString &NodeName, uint32 SampleRate, uint16 Channels, uint16 BitsPerSample)> Callback
) 
```



TTS audio format callback. Fires once per turn, before any OnTtsAudio. Parameters: (NodeName, SampleRate, Channels, BitsPerSample). NodeName is the producing node's name (multi-sender attribution — mirrors OnAnswerText). Only mono int16 PCM is currently emitted by the server. 


        

<hr>



### function SetOnTurnComplete 

```C++
void FTryllAgent::SetOnTurnComplete (
    TFunction< void(ETryllTurnStatus Status, const FString &DebugInfo, int32 TokensGenerated)> Callback
) 
```



Called once per turn after all AnswerText chunks, carrying the turn outcome. TokensGenerated: server-reported total tokens sampled across all generation nodes in this turn (prompt tokens excluded). 


        

<hr>



### function UnregisterTool 

```C++
bool FTryllAgent::UnregisterTool (
    const FString & ToolName
) 
```



Remove a previously registered tool handler. 


        

<hr>



### function WasCreatedWithDiagnostics 

```C++
inline bool FTryllAgent::WasCreatedWithDiagnostics () noexcept const
```




<hr>



### function ~FTryllAgent 

```C++
FTryllAgent::~FTryllAgent () 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllAgent.h`

