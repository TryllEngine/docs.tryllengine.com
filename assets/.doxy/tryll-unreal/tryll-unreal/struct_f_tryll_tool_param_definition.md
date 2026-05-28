

# Struct FTryllToolParamDefinition



[**ClassList**](annotated.md) **>** [**FTryllToolParamDefinition**](struct_f_tryll_tool_param_definition.md)



[More...](#detailed-description)

* `#include <TryllToolParamDefinition.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  bool | [**bOverrideDescription**](#variable-boverridedescription)   = `false`<br> |
|  bool | [**bOverrideName**](#variable-boverridename)   = `false`<br> |
|  bool | [**bOverrideType**](#variable-boverridetype)   = `false`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**UPROPERTY**](#function-uproperty-13) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|ToolParamDefinition", meta=(EditCondition="bOverrideName")) <br> |
|   | [**UPROPERTY**](#function-uproperty-23) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|ToolParamDefinition", meta=(EditCondition="bOverrideType")) <br> |
|   | [**UPROPERTY**](#function-uproperty-33) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|ToolParamDefinition", meta=(EditCondition="bOverrideDescription")) <br> |




























## Detailed Description


One named parameter in a tool's schema (used in SLM prompt construction). 


    
## Public Attributes Documentation




### variable bOverrideDescription 

```C++
bool FTryllToolParamDefinition::bOverrideDescription;
```



Human-readable parameter description for the SLM prompt. 


        

<hr>



### variable bOverrideName 

```C++
bool FTryllToolParamDefinition::bOverrideName;
```



Parameter name exposed to the SLM in the tool schema. 


        

<hr>



### variable bOverrideType 

```C++
bool FTryllToolParamDefinition::bOverrideType;
```



JSON-schema-style type string (e.g. "string", "number"). 


        

<hr>
## Public Functions Documentation




### function UPROPERTY [1/3]

```C++
FTryllToolParamDefinition::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|ToolParamDefinition",
    meta=(EditCondition="bOverrideName")
) 
```




<hr>



### function UPROPERTY [2/3]

```C++
FTryllToolParamDefinition::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|ToolParamDefinition",
    meta=(EditCondition="bOverrideType")
) 
```




<hr>



### function UPROPERTY [3/3]

```C++
FTryllToolParamDefinition::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|ToolParamDefinition",
    meta=(EditCondition="bOverrideDescription")
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/Generated/Nodes/TryllToolParamDefinition.h`

