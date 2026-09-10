

# Struct FTryllToolCall



[**ClassList**](annotated.md) **>** [**FTryllToolCall**](struct_f_tryll_tool_call.md)


























## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**ArgumentsJson**](#variable-argumentsjson)  <br> |
|  FString | [**CallId**](#variable-callid)  <br> |
|  FString | [**Disposition**](#variable-disposition)  <br> |
|  FString | [**NodeName**](#variable-nodename)  <br> |
|  FString | [**ToolName**](#variable-toolname)  <br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  bool | [**RequiresNativeResults**](#function-requiresnativeresults) () const<br> |




























## Public Attributes Documentation




### variable ArgumentsJson 

```C++
FString FTryllToolCall::ArgumentsJson;
```




<hr>



### variable CallId 

```C++
FString FTryllToolCall::CallId;
```




<hr>



### variable Disposition 

```C++
FString FTryllToolCall::Disposition;
```



Snapshot of disposition ("route\_only" / "acknowledge" / "notify" / "notify\_and\_acknowledge" / "pause" / "pause\_and\_acknowledge" / "await\_result"). RegisterTool uses this to decide whether auto-resume must carry tool results (await\_result only). 


        

<hr>



### variable NodeName 

```C++
FString FTryllToolCall::NodeName;
```




<hr>



### variable ToolName 

```C++
FString FTryllToolCall::ToolName;
```




<hr>
## Public Functions Documentation




### function RequiresNativeResults 

```C++
inline bool FTryllToolCall::RequiresNativeResults () const
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllToolCall.h`

