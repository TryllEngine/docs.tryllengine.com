

# File TryllVariableDecl.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllVariableDecl.h**](_tryll_variable_decl_8h.md)





* `#include "CoreMinimal.h"`
* `#include "TryllVariableDecl.generated.h"`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) <br> |
| struct | [**FTryllVariableOverride**](struct_f_tryll_variable_override.md) <br> |
| struct | [**FTryllVariableUpdate**](struct_f_tryll_variable_update.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| enum uint8 | [**ETryllVariableOp**](#enum-etryllvariableop)  <br> |
| enum uint8 | [**ETryllVariableType**](#enum-etryllvariabletype)  <br> |
















































## Public Types Documentation




### enum ETryllVariableOp 

```C++
enum ETryllVariableOp {
    Assign = 0,
    AddToSet = 1,
    RemoveFromSet = 2,
    Reset = 3
};
```



Mutation applied to one declared agent variable in a batched UpdateAgentVariablesRequest. Mirrors Tryll::VariableOp (wire enum) — ordinal values must stay in sync.


Assign: overwrite; value must match the variable's declared type. AddToSet/RemoveFromSet: target must be StringSet; value carried in StringValue (idempotent — a no-op success if already present/absent). Reset: restore the CreateAgent-time initial value; no value fields are read. 


        

<hr>



### enum ETryllVariableType 

```C++
enum ETryllVariableType {
    Int = 0,
    Float = 1,
    String = 2,
    Bool = 3,
    StringSet = 4
};
```



Wire type of one declared per-agent variable. Mirrors the Tryll::VariableValue union member that carries the value (VarInt/VarFloat/VarString/VarBool/VarStringSet). 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllVariableDecl.h`

