

# Class UTryllAgentComponent



[**ClassList**](annotated.md) **>** [**UTryllAgentComponent**](class_u_tryll_agent_component.md)



[More...](#detailed-description)

* `#include <TryllAgentComponent.h>`



Inherits the following classes: UActorComponent


















## Public Attributes

| Type | Name |
| ---: | :--- |
|  [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) | [**InlineGraphDescription**](#variable-inlinegraphdescription)  <br> |
|  FOnTryllAgentReady | [**OnAgentReady**](#variable-onagentready)  <br> |
|  FOnTryllAgentAnswerFull | [**OnAnswerFull**](#variable-onanswerfull)  <br> |
|  FOnTryllAgentAnswerText | [**OnAnswerText**](#variable-onanswertext)  <br> |
|  FOnTryllAgentError | [**OnError**](#variable-onerror)  <br> |
|  FOnTryllParamChanged | [**OnParamChanged**](#variable-onparamchanged)  <br> |
|  FOnTryllAgentTurnComplete | [**OnTurnComplete**](#variable-onturncomplete)  <br> |
|  TObjectPtr&lt; [**UTryllSpeakerComponent**](class_u_tryll_speaker_component.md) &gt; | [**Speaker**](#variable-speaker)   = `nullptr`<br> |
|  [**UTryllWorkflowAsset**](class_u_tryll_workflow_asset.md) \* | [**WorkflowAsset**](#variable-workflowasset)   = `nullptr`<br> |
|  bool | [**bAutoCreateOnConnect**](#variable-bautocreateonconnect)   = `true`<br> |
|  bool | [**bEnableDiagnostics**](#variable-benablediagnostics)   = `false`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  void | [**ChangeParams**](#function-changeparams) (const FString & NodeName, [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* Params) <br> |
|  void | [**CreateAgent**](#function-createagent) () <br> |
|  void | [**DestroyAgent**](#function-destroyagent) () <br> |
|  int64 | [**GetAgentId**](#function-getagentid) () const<br> |
|  [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* | [**GetNodeParamsBaseline**](#function-getnodeparamsbaseline) (const FString & NodeName) const<br> |
|  TArray&lt; FString &gt; | [**GetStartNodeOptions**](#function-getstartnodeoptions) () const<br> |
|  bool | [**IsAgentReady**](#function-isagentready) () const<br> |
|  void | [**SendMessage**](#function-sendmessage) (const FString & Message) <br> |
|   | [**UTryllAgentComponent**](#function-utryllagentcomponent) () <br> |
























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




The component auto-creates an agent once the session is ready (if bAutoCreateOnConnect) — it queues until OnConfigureSessionComplete if it spawns before the session is configured. It auto-destroys the agent on EndPlay. 


    
## Public Attributes Documentation




### variable InlineGraphDescription 

```C++
FTryllGraphDescription UTryllAgentComponent::InlineGraphDescription;
```



Fallback graph description used when WorkflowAsset is not set. Can be populated programmatically via FTryllGraphBuilder::Build(). 


        

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



### variable OnError 

```C++
FOnTryllAgentError UTryllAgentComponent::OnError;
```



Fired on any agent-level error (send failure, server error response). 


        

<hr>



### variable OnParamChanged 

```C++
FOnTryllParamChanged UTryllAgentComponent::OnParamChanged;
```



Fired when a ChangeParam request completes.
* bSuccess = true on Ack; false on any server error.
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
## Public Functions Documentation




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



### function GetAgentId 

```C++
inline int64 UTryllAgentComponent::GetAgentId () const
```



Server-assigned agent id, or 0 if the agent has not been created yet. 


        

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



### function IsAgentReady 

```C++
inline bool UTryllAgentComponent::IsAgentReady () const
```



True after the server has confirmed agent creation (OnAgentReady has fired). 


        

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

