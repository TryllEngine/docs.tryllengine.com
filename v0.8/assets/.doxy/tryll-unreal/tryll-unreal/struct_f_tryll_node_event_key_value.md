

# Struct FTryllNodeEventKeyValue



[**ClassList**](annotated.md) **>** [**FTryllNodeEventKeyValue**](struct_f_tryll_node_event_key_value.md)



[More...](#detailed-description)

* `#include <TryllNodeEvent.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Key**](#variable-key)  <br> |
|  FString | [**Value**](#variable-value)  <br> |












































## Detailed Description


One key/value from a NodeEvent payload. Order matches the wire. Shared by the session fallback ([**UTryllSubsystem::OnNodeEvent**](class_u_tryll_subsystem.md#variable-onnodeevent)) and the per-agent fallback ([**UTryllAgentComponent::OnNodeEvent**](class_u_tryll_agent_component.md#variable-onnodeevent)). 


    
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
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllNodeEvent.h`

