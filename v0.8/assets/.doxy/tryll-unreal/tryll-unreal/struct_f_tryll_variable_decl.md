

# Struct FTryllVariableDecl



[**ClassList**](annotated.md) **>** [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md)



[More...](#detailed-description)

* `#include <TryllVariableDecl.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  double | [**FloatValue**](#variable-floatvalue)   = `0.0`<br> |
|  int64 | [**IntValue**](#variable-intvalue)   = `0`<br> |
|  FString | [**Name**](#variable-name)  <br> |
|  TArray&lt; FString &gt; | [**StringSetValue**](#variable-stringsetvalue)  <br> |
|  FString | [**StringValue**](#variable-stringvalue)  <br> |
|  ETryllVariableType | [**Type**](#variable-type)   = `ETryllVariableType::Int`<br> |
|  bool | [**bAllowOutputSubstitution**](#variable-ballowoutputsubstitution)   = `false`<br> |
|  bool | [**bBoolValue**](#variable-bboolvalue)   = `false`<br> |












































## Detailed Description


One declared per-agent variable — name, type, and initial value. Authored on [**UTryllWorkflowAsset::Variables**](class_u_tryll_workflow_asset.md#variable-variables); the complete list is sent once in CreateAgentRequest and locks each variable's type for the agent's lifetime. Only one of the value fields is meaningful, selected by Type — the Details customization shows only that field. 


    
## Public Attributes Documentation




### variable FloatValue 

```C++
double FTryllVariableDecl::FloatValue;
```




<hr>



### variable IntValue 

```C++
int64 FTryllVariableDecl::IntValue;
```




<hr>



### variable Name 

```C++
FString FTryllVariableDecl::Name;
```




<hr>



### variable StringSetValue 

```C++
TArray<FString> FTryllVariableDecl::StringSetValue;
```




<hr>



### variable StringValue 

```C++
FString FTryllVariableDecl::StringValue;
```




<hr>



### variable Type 

```C++
ETryllVariableType FTryllVariableDecl::Type;
```




<hr>



### variable bAllowOutputSubstitution 

```C++
bool FTryllVariableDecl::bAllowOutputSubstitution;
```




<hr>



### variable bBoolValue 

```C++
bool FTryllVariableDecl::bBoolValue;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllVariableDecl.h`

