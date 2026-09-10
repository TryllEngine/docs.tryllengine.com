

# Struct FTryllVariableOverride



[**ClassList**](annotated.md) **>** [**FTryllVariableOverride**](struct_f_tryll_variable_override.md)



[More...](#detailed-description)

* `#include <TryllVariableDecl.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Name**](#variable-name)  <br> |
|  [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) | [**Value**](#variable-value)  <br> |
|  bool | [**bEnabled**](#variable-benabled)   = `false`<br> |












































## Detailed Description


Per-component value override for one of the workflow asset's declared variables (Material-Instance pattern: the asset owns the declaration — name and type — the component may only override the initial value). Name and Value.Type must match a variable declared on the asset; non-matching (stale) overrides are skipped with a warning at agent-creation time. 


    
## Public Attributes Documentation




### variable Name 

```C++
FString FTryllVariableOverride::Name;
```




<hr>



### variable Value 

```C++
FTryllVariableDecl FTryllVariableOverride::Value;
```




<hr>



### variable bEnabled 

```C++
bool FTryllVariableOverride::bEnabled;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllVariableDecl.h`

