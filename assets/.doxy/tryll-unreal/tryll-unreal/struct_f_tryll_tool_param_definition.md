

# Struct FTryllToolParamDefinition



[**ClassList**](annotated.md) **>** [**FTryllToolParamDefinition**](struct_f_tryll_tool_param_definition.md)



[More...](#detailed-description)

* `#include <TryllToolParamDefinition.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Description**](#variable-description)  <br> |
|  FString | [**Name**](#variable-name)  <br> |
|  FString | [**Type**](#variable-type)  <br> |
|  bool | [**bOverrideDescription**](#variable-boverridedescription)   = `false`<br> |
|  bool | [**bOverrideName**](#variable-boverridename)   = `false`<br> |
|  bool | [**bOverrideType**](#variable-boverridetype)   = `false`<br> |












































## Detailed Description


One named parameter in a tool's schema (used in SLM prompt construction). 


    
## Public Attributes Documentation




### variable Description 

```C++
FString FTryllToolParamDefinition::Description;
```




<hr>



### variable Name 

```C++
FString FTryllToolParamDefinition::Name;
```




<hr>



### variable Type 

```C++
FString FTryllToolParamDefinition::Type;
```




<hr>



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

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllToolParamDefinition.h`

