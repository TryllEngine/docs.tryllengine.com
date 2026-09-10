

# File TryllToolCall.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllToolCall.h**](_tryll_tool_call_8h.md)





* `#include "CoreMinimal.h"`
* `#include "TryllToolCall.generated.h"`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTryllToolCall**](struct_f_tryll_tool_call.md) <br> |
| struct | [**FTryllToolHandlerResult**](struct_f_tryll_tool_handler_result.md) <br> |
| struct | [**FTryllToolResult**](struct_f_tryll_tool_result.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| typedef TFunction&lt; FString(const [**FTryllToolCall**](struct_f_tryll_tool_call.md) &Call)&gt; | [**FTryllSimpleToolHandler**](#typedef-ftryllsimpletoolhandler)  <br> |
| typedef TFunction&lt; [**FTryllToolHandlerResult**](struct_f_tryll_tool_handler_result.md)(const [**FTryllToolCall**](struct_f_tryll_tool_call.md) &Call)&gt; | [**FTryllToolHandler**](#typedef-ftrylltoolhandler)  <br> |
















































## Public Types Documentation




### typedef FTryllSimpleToolHandler 

```C++
using FTryllSimpleToolHandler = 
TFunction<FString(const FTryllToolCall& Call)>;
```




<hr>



### typedef FTryllToolHandler 

```C++
using FTryllToolHandler = 
TFunction<FTryllToolHandlerResult(const FTryllToolCall& Call)>;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllToolCall.h`

