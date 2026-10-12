

# Struct FTryllToolHandlerResult



[**ClassList**](annotated.md) **>** [**FTryllToolHandlerResult**](struct_f_tryll_tool_handler_result.md)



[More...](#detailed-description)

* `#include <TryllToolCall.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Error**](#variable-error)  <br> |
|  FString | [**Result**](#variable-result)  <br> |
|  bool | [**bSuccess**](#variable-bsuccess)   = `true`<br> |


















## Public Static Functions

| Type | Name |
| ---: | :--- |
|  [**FTryllToolHandlerResult**](struct_f_tryll_tool_handler_result.md) | [**Failure**](#function-failure) (FString InError) <br> |
|  [**FTryllToolHandlerResult**](struct_f_tryll_tool_handler_result.md) | [**Success**](#function-success) (FString InResult={}) <br> |


























## Detailed Description


Outcome returned by a registered native tool handler. 


    
## Public Attributes Documentation




### variable Error 

```C++
FString FTryllToolHandlerResult::Error;
```




<hr>



### variable Result 

```C++
FString FTryllToolHandlerResult::Result;
```




<hr>



### variable bSuccess 

```C++
bool FTryllToolHandlerResult::bSuccess;
```




<hr>
## Public Static Functions Documentation




### function Failure 

```C++
static inline FTryllToolHandlerResult FTryllToolHandlerResult::Failure (
    FString InError
) 
```




<hr>



### function Success 

```C++
static inline FTryllToolHandlerResult FTryllToolHandlerResult::Success (
    FString InResult={}
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllToolCall.h`

