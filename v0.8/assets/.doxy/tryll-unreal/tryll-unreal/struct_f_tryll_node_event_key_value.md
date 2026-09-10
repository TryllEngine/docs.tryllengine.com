

# Struct FTryllNodeEventKeyValue



[**ClassList**](annotated.md) **>** [**FTryllNodeEventKeyValue**](struct_f_tryll_node_event_key_value.md)



_Embedding ClassifyIntent — event\_type="intent\_classified". Subscribing swallows OnNodeEvent._ [More...](#detailed-description)

* `#include <TryllSubsystem.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Key**](#variable-key)  <br> |
|  FString | [**Value**](#variable-value)  <br> |












































## Detailed Description


Detailed tool-call event carrying the full owning [**FTryllToolCall**](struct_f_tryll_tool_call.md) value object (node name and disposition included) alongside the agent id. Fired in addition to — never instead of — OnToolCall / OnToolCallWithId, so a manual handler can tell whether to send results for await\_result or plain-resume for other pausing dispositions. ClassifyIntentLLM — event\_type="intent\_llm\_classified". Subscribing swallows OnNodeEvent. Pause typed event — fired for NodeEvent event\_type="paused" (the executor paused the turn between nodes; ChangeAgentParam is allowed while paused). Generic NodeEvent fallback — fired for any event\_type without a typed delegate (or when the typed delegate has no subscribers). KvPairs preserves wire order. 


    
## Public Attributes Documentation




### variable Key 

```C++
FString FTryllNodeEventKeyValue::Key;
```




<hr>



### variable Value 

```C++
FString FTryllNodeEventKeyValue::Value;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllSubsystem.h`

