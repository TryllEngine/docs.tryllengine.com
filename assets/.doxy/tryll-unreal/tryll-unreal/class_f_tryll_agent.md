

# Class FTryllAgent



[**ClassList**](annotated.md) **>** [**FTryllAgent**](class_f_tryll_agent.md)



[More...](#detailed-description)

* `#include <TryllAgent.h>`



Inherits the following classes: TSharedFromThis< FTryllAgent >


































## Public Functions

| Type | Name |
| ---: | :--- |
|  void | [**ChangeParams**](#function-changeparams) (const FString & NodeName, [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* Params, TFunction&lt; void(const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete=nullptr) <br> |
|  void | [**ClearAfterDestroyRequest**](#function-clearafterdestroyrequest) () <br> |
|   | [**FTryllAgent**](#function-ftryllagent) (TSharedPtr&lt; FTryllConnection &gt; InConnection, std::uint64\_t InAgentId) <br> |
|  std::uint64\_t | [**GetAgentId**](#function-getagentid) () noexcept const<br> |
|  [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* | [**GetNodeParamsBaseline**](#function-getnodeparamsbaseline) (const FString & NodeName) const<br> |
|  void | [**HandleAnswerText**](#function-handleanswertext) (const FString & Text, bool bIsDelta, bool bIsFinal) <br> |
|  void | [**HandleChangeParamResult**](#function-handlechangeparamresult) (std::uint64\_t RequestId, const [**FTryllError**](struct_f_tryll_error.md) & Error) <br> |
|  void | [**HandleError**](#function-handleerror) (const [**FTryllError**](struct_f_tryll_error.md) & Error) <br> |
|  void | [**HandleTurnComplete**](#function-handleturncomplete) (ETryllTurnStatus Status, const FString & DebugInfo, int32 TokensGenerated) <br> |
|  bool | [**IsValid**](#function-isvalid) () noexcept const<br> |
|  void | [**SendMessage**](#function-sendmessage) (const FString & Text) <br> |
|  void | [**SetNodeParamsBaseline**](#function-setnodeparamsbaseline) (const FString & NodeName, [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* Params) <br> |
|  void | [**SetOnAnswerText**](#function-setonanswertext) (TFunction&lt; void(const FString &Text, bool bIsDelta, bool bIsFinal)&gt; Callback) <br> |
|  void | [**SetOnError**](#function-setonerror) (TFunction&lt; void(const [**FTryllError**](struct_f_tryll_error.md) &Error)&gt; Callback) <br> |
|  void | [**SetOnTurnComplete**](#function-setonturncomplete) (TFunction&lt; void(ETryllTurnStatus Status, const FString &DebugInfo, int32 TokensGenerated)&gt; Callback) <br> |
|   | [**~FTryllAgent**](#function-ftryllagent) () <br> |




























## Detailed Description


Plain C++ handle for a server-side agent. Created by [**UTryllSubsystem**](class_u_tryll_subsystem.md) after a successful CreateAgentResponse.


Ownership: typically held as TSharedPtr&lt;FTryllAgent&gt; by UTryllAgentComponent. Lifetime: deterministic — the owning component releases the shared\_ptr; this triggers PushDestroyAgent via [**UTryllSubsystem::RequestDestroyAgent()**](class_u_tryll_subsystem.md#function-requestdestroyagent).


All callbacks are invoked on the game thread (from UTryllSubsystem::Tick). 


    
## Public Functions Documentation




### function ChangeParams 

```C++
void FTryllAgent::ChangeParams (
    const FString & NodeName,
    UTryllNodeParamsBase * Params,
    TFunction< void(const FTryllError &)> OnComplete=nullptr
) 
```



Apply typed node parameters to a workflow node at runtime. The agent must not be processing a turn. Completion is invoked on the game thread via the OnComplete callback.


Clone-set-send idiom (C++): auto\* P = UTryllNodeParamsFactory::CloneParams(AgentComp-&gt;GetNodeParamsBaseline("gen"), this); Cast&lt;UTryllGenerateParams&gt;(P)-&gt;SystemPrompt = TEXT("New prompt"); Agent-&gt;ChangeParams(TEXT("gen"), P); 


        

<hr>



### function ClearAfterDestroyRequest 

```C++
void FTryllAgent::ClearAfterDestroyRequest () 
```



Called by [**UTryllSubsystem**](class_u_tryll_subsystem.md) when the destroy Ack arrives; clears the local agent id. 


        

<hr>



### function FTryllAgent 

```C++
FTryllAgent::FTryllAgent (
    TSharedPtr< FTryllConnection > InConnection,
    std::uint64_t InAgentId
) 
```




<hr>



### function GetAgentId 

```C++
inline std::uint64_t FTryllAgent::GetAgentId () noexcept const
```




<hr>



### function GetNodeParamsBaseline 

```C++
UTryllNodeParamsBase * FTryllAgent::GetNodeParamsBaseline (
    const FString & NodeName
) const
```



Return a deep copy of the last authored params for the named node. Use this as the starting point for a clone-set-send mutation; structural fields are automatically propagated from the baseline. 


        

<hr>



### function HandleAnswerText 

```C++
void FTryllAgent::HandleAnswerText (
    const FString & Text,
    bool bIsDelta,
    bool bIsFinal
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



### function HandleTurnComplete 

```C++
void FTryllAgent::HandleTurnComplete (
    ETryllTurnStatus Status,
    const FString & DebugInfo,
    int32 TokensGenerated
) 
```




<hr>



### function IsValid 

```C++
inline bool FTryllAgent::IsValid () noexcept const
```




<hr>



### function SendMessage 

```C++
void FTryllAgent::SendMessage (
    const FString & Text
) 
```



Send a text message to the agent. Response arrives via the OnAnswerText / OnTurnComplete callbacks. 


        

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
    TFunction< void(const FString &Text, bool bIsDelta, bool bIsFinal)> Callback
) 
```



Streaming text callback. Matches the current AnswerText wire message:
* bIsDelta = true → Text is an incremental token/chunk.
* bIsDelta = false → Text is the full response accumulated so far.
* bIsFinal = true → This is the last AnswerText before TurnComplete. 




        

<hr>



### function SetOnError 

```C++
void FTryllAgent::SetOnError (
    TFunction< void(const FTryllError &Error)> Callback
) 
```



Called on any send-side error (connection loss, server error response, etc.). 


        

<hr>



### function SetOnTurnComplete 

```C++
void FTryllAgent::SetOnTurnComplete (
    TFunction< void(ETryllTurnStatus Status, const FString &DebugInfo, int32 TokensGenerated)> Callback
) 
```



Called once per turn after all AnswerText chunks, carrying the turn outcome. TokensGenerated: server-reported total tokens sampled across all generation nodes in this turn (prompt tokens excluded). 


        

<hr>



### function ~FTryllAgent 

```C++
FTryllAgent::~FTryllAgent () 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/TryllAgent.h`

