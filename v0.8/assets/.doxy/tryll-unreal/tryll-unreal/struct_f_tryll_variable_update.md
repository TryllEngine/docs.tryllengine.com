

# Struct FTryllVariableUpdate



[**ClassList**](annotated.md) **>** [**FTryllVariableUpdate**](struct_f_tryll_variable_update.md)



[More...](#detailed-description)

* `#include <TryllVariableDecl.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  double | [**FloatValue**](#variable-floatvalue)   = `0.0`<br> |
|  int64 | [**IntValue**](#variable-intvalue)   = `0`<br> |
|  FString | [**Name**](#variable-name)  <br> |
|  ETryllVariableOp | [**Op**](#variable-op)   = `ETryllVariableOp::Assign`<br> |
|  TArray&lt; FString &gt; | [**StringSetValue**](#variable-stringsetvalue)  <br> |
|  FString | [**StringValue**](#variable-stringvalue)  <br> |
|  ETryllVariableType | [**ValueType**](#variable-valuetype)   = `ETryllVariableType::Int`<br> |
|  bool | [**bBoolValue**](#variable-bboolvalue)   = `false`<br> |












































## Detailed Description


One staged mutation, queued by [**FTryllAgentVariables**](class_f_tryll_agent_variables.md) and flushed as a single UpdateAgentVariablesRequest. Plain internal plumbing (not a USTRUCT — never exposed to Blueprint/editor). 


    
## Public Attributes Documentation




### variable FloatValue 

```C++
double FTryllVariableUpdate::FloatValue;
```




<hr>



### variable IntValue 

```C++
int64 FTryllVariableUpdate::IntValue;
```




<hr>



### variable Name 

```C++
FString FTryllVariableUpdate::Name;
```




<hr>



### variable Op 

```C++
ETryllVariableOp FTryllVariableUpdate::Op;
```




<hr>



### variable StringSetValue 

```C++
TArray<FString> FTryllVariableUpdate::StringSetValue;
```




<hr>



### variable StringValue 

```C++
FString FTryllVariableUpdate::StringValue;
```




<hr>



### variable ValueType 

```C++
ETryllVariableType FTryllVariableUpdate::ValueType;
```




<hr>



### variable bBoolValue 

```C++
bool FTryllVariableUpdate::bBoolValue;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllVariableDecl.h`

