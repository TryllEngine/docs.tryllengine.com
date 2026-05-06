

# Struct FTryllNodeDescription



[**ClassList**](annotated.md) **>** [**FTryllNodeDescription**](struct_f_tryll_node_description.md)



[More...](#detailed-description)

* `#include <TryllGraphDescription.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Name**](#variable-name)  <br> |
|  TMap&lt; FString, FString &gt; | [**Params**](#variable-params)  <br> |
|  TArray&lt; [**FTryllToolDefinition**](struct_f_tryll_tool_definition.md) &gt; | [**Tools**](#variable-tools)  <br> |
|  ETryllNodeType | [**Type**](#variable-type)   = `ETryllNodeType::Generate`<br> |












































## Detailed Description


Description of one workflow node sent to the server. 


    
## Public Attributes Documentation




### variable Name 

```C++
FString FTryllNodeDescription::Name;
```




<hr>



### variable Params 

```C++
TMap<FString, FString> FTryllNodeDescription::Params;
```



String key/value parameters passed to the node (e.g. "model\_name", "system\_prompt"). Using TMap for convenience; serialized as a flat vector of NodeParam in the FlatBuffers payload. 


        

<hr>



### variable Tools 

```C++
TArray<FTryllToolDefinition> FTryllNodeDescription::Tools;
```



Tool definitions for ToolCall nodes. Ignored for all other node types. 


        

<hr>



### variable Type 

```C++
ETryllNodeType FTryllNodeDescription::Type;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo/server/client-unreal/Source/TryllClient/Public/TryllGraphDescription.h`

