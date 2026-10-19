

# Struct FTryllDecodedNodeEvent



[**ClassList**](annotated.md) **>** [**FTryllDecodedNodeEvent**](struct_f_tryll_decoded_node_event.md)



[More...](#detailed-description)

* `#include <TryllNodeEvent.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**EventType**](#variable-eventtype)  <br> |
|  [**FTryllIntentClassification**](struct_f_tryll_intent_classification.md) | [**Intent**](#variable-intent)  <br> |
|  [**FTryllIntentLlmClassification**](struct_f_tryll_intent_llm_classification.md) | [**IntentLlm**](#variable-intentllm)  <br> |
|  ETryllNodeEventKind | [**Kind**](#variable-kind)   = `ETryllNodeEventKind::Unknown`<br> |
|  TArray&lt; [**FTryllNodeEventKeyValue**](struct_f_tryll_node_event_key_value.md) &gt; | [**KvPairs**](#variable-kvpairs)  <br> |
|  FString | [**NodeName**](#variable-nodename)  <br> |
|  FString | [**PendingExit**](#variable-pendingexit)  <br> |
|  [**FTryllToolCall**](struct_f_tryll_tool_call.md) | [**ToolCall**](#variable-toolcall)  <br> |












































## Detailed Description


Owning decode of one NodeEvent. Strings outlive the inbound frame. 


    
## Public Attributes Documentation




### variable EventType 

```C++
FString FTryllDecodedNodeEvent::EventType;
```




<hr>



### variable Intent 

```C++
FTryllIntentClassification FTryllDecodedNodeEvent::Intent;
```




<hr>



### variable IntentLlm 

```C++
FTryllIntentLlmClassification FTryllDecodedNodeEvent::IntentLlm;
```




<hr>



### variable Kind 

```C++
ETryllNodeEventKind FTryllDecodedNodeEvent::Kind;
```




<hr>



### variable KvPairs 

```C++
TArray<FTryllNodeEventKeyValue> FTryllDecodedNodeEvent::KvPairs;
```




<hr>



### variable NodeName 

```C++
FString FTryllDecodedNodeEvent::NodeName;
```




<hr>



### variable PendingExit 

```C++
FString FTryllDecodedNodeEvent::PendingExit;
```



Pause only: the paused node's pending exit route (wire key "exit\_route"). 


        

<hr>



### variable ToolCall 

```C++
FTryllToolCall FTryllDecodedNodeEvent::ToolCall;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllNodeEvent.h`

