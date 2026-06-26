

# Struct FTryllNodeEventKeyValue



[**ClassList**](annotated.md) **>** [**FTryllNodeEventKeyValue**](struct_f_tryll_node_event_key_value.md)



_Intent classification typed event — fired for NodeEvent event\_type="intent\_classified"._ [More...](#detailed-description)

* `#include <TryllSubsystem.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Key**](#variable-key)  <br> |
|  FString | [**Value**](#variable-value)  <br> |












































## Detailed Description


Generic NodeEvent fallback — fired for any event\_type without a typed delegate (or when the typed delegate has no subscribers). KvPairs preserves wire order. 


    
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

