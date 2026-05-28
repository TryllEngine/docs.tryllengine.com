

# Struct FTryllToolDefinition



[**ClassList**](annotated.md) **>** [**FTryllToolDefinition**](struct_f_tryll_tool_definition.md)



[More...](#detailed-description)

* `#include <TryllToolDefinition.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  bool | [**bOverrideDescription**](#variable-boverridedescription)   = `false`<br> |
|  bool | [**bOverrideName**](#variable-boverridename)   = `false`<br> |
|  bool | [**bOverrideParameters**](#variable-boverrideparameters)   = `false`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**UPROPERTY**](#function-uproperty-13) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|ToolDefinition", meta=(EditCondition="bOverrideName")) <br> |
|   | [**UPROPERTY**](#function-uproperty-23) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|ToolDefinition", meta=(EditCondition="bOverrideDescription")) <br> |
|   | [**UPROPERTY**](#function-uproperty-33) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|ToolDefinition", meta=(EditCondition="bOverrideParameters")) <br> |




























## Detailed Description


A callable tool definition used for prompt construction by ToolCallNode. The server does not execute tools; these are schema-only for SLM prompting. 


    
## Public Attributes Documentation




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



### variable bOverrideParameters 

```C++
bool FTryllToolDefinition::bOverrideParameters;
```



Named parameters accepted by this tool. 


        

<hr>
## Public Functions Documentation




### function UPROPERTY [1/3]

```C++
FTryllToolDefinition::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|ToolDefinition",
    meta=(EditCondition="bOverrideName")
) 
```




<hr>



### function UPROPERTY [2/3]

```C++
FTryllToolDefinition::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|ToolDefinition",
    meta=(EditCondition="bOverrideDescription")
) 
```




<hr>



### function UPROPERTY [3/3]

```C++
FTryllToolDefinition::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|ToolDefinition",
    meta=(EditCondition="bOverrideParameters")
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/Generated/Nodes/TryllToolDefinition.h`

