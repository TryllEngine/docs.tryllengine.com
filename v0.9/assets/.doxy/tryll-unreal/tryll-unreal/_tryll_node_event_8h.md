

# File TryllNodeEvent.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllNodeEvent.h**](_tryll_node_event_8h.md)





* `#include "CoreMinimal.h"`
* `#include "TryllIntentClassification.h"`
* `#include "TryllToolCall.h"`
* `#include "TryllNodeEvent.generated.h"`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTryllDecodedNodeEvent**](struct_f_tryll_decoded_node_event.md) <br> |
| struct | [**FTryllNodeEventKeyValue**](struct_f_tryll_node_event_key_value.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| enum uint8 | [**ETryllAgentNodeEventKind**](#enum-etryllagentnodeeventkind)  <br> |
| enum uint8 | [**ETryllNodeEventKind**](#enum-etryllnodeeventkind)  <br> |




















## Public Functions

| Type | Name |
| ---: | :--- |
|  [**FTryllDecodedNodeEvent**](struct_f_tryll_decoded_node_event.md) | [**TryllDecodeNodeEvent**](#function-trylldecodenodeevent) (const FString & NodeName, const FString & EventType, const TArray&lt; TPair&lt; FString, FString &gt; &gt; & WirePairs) <br> |




























## Public Types Documentation




### enum ETryllAgentNodeEventKind 

```C++
enum ETryllAgentNodeEventKind {
    ToolCall,
    ToolCallWithId,
    ToolCallDetailed,
    IntentClassified,
    IntentLlmClassified,
    Paused,
    NodeEvent
};
```



Listener slots on one agent. The component's probe reports these from Blueprint IsBound(); a raw [**FTryllAgent**](class_f_tryll_agent.md) reports them from its callback slots. ToolCallDetailed suppresses the agent's generic OnNodeEvent (C++ AgentProxy parity) even though the session-level detailed event does not. 


        

<hr>



### enum ETryllNodeEventKind 

```C++
enum ETryllNodeEventKind {
    ToolCall,
    IntentClassified,
    IntentLlmClassified,
    Paused,
    Unknown
};
```



Which typed NodeEvent the shared decoder recognised. 


        

<hr>
## Public Functions Documentation




### function TryllDecodeNodeEvent 

```C++
FTryllDecodedNodeEvent TryllDecodeNodeEvent (
    const FString & NodeName,
    const FString & EventType,
    const TArray< TPair< FString, FString > > & WirePairs
) 
```



Parse a NodeEvent payload once. Both dispatch sites ([**UTryllSubsystem**](class_u_tryll_subsystem.md) and UTryllEditorSubsystem) call this, then broadcast their own session delegates and hand the result to [**FTryllAgent::HandleNodeEvent**](class_f_tryll_agent.md#function-handlenodeevent). 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllNodeEvent.h`

