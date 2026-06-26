

# Struct FTryllToolDefinition



[**ClassList**](annotated.md) **>** [**FTryllToolDefinition**](struct_f_tryll_tool_definition.md)



[More...](#detailed-description)

* `#include <TryllToolDefinition.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Description**](#variable-description)  <br> |
|  FString | [**Name**](#variable-name)  <br> |
|  TArray&lt; [**FTryllToolParamDefinition**](struct_f_tryll_tool_param_definition.md) &gt; | [**Parameters**](#variable-parameters)  <br> |
|  bool | [**bOverrideDescription**](#variable-boverridedescription)   = `false`<br> |
|  bool | [**bOverrideName**](#variable-boverridename)   = `false`<br> |












































## Detailed Description


A callable tool definition used for prompt construction by ToolCallNode. The server does not execute tools; these are schema-only for SLM prompting. 


    
## Public Attributes Documentation




### variable Description 

```C++
FString FTryllToolDefinition::Description;
```




<hr>



### variable Name 

```C++
FString FTryllToolDefinition::Name;
```




<hr>



### variable Parameters 

```C++
TArray<FTryllToolParamDefinition> FTryllToolDefinition::Parameters;
```



Named parameters accepted by this tool. 


        

<hr>



### variable bOverrideDescription 

```C++
bool FTryllToolDefinition::bOverrideDescription;
```



What the tool does; included in the SLM tool-call prompt. 


        

<hr>



### variable bOverrideName 

```C++
bool FTryllToolDefinition::bOverrideName;
```



Tool name the SLM must use when calling this tool. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllToolDefinition.h`

