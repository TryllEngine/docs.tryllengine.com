

# Class UTryllAgentVariables



[**ClassList**](annotated.md) **>** [**UTryllAgentVariables**](class_u_tryll_agent_variables.md)



[More...](#detailed-description)

* `#include <TryllAgentVariables.h>`



Inherits the following classes: UObject


































## Public Functions

| Type | Name |
| ---: | :--- |
|  [**FTryllError**](struct_f_tryll_error.md) | [**AddToSetVariable**](#function-addtosetvariable) (const FString & Name, const FString & Element) <br> |
|  void | [**FlushIfDirty**](#function-flushifdirty) () <br> |
|  bool | [**GetBoolVariable**](#function-getboolvariable) (const FString & Name, bool & bFound) const<br> |
|  float | [**GetFloatVariable**](#function-getfloatvariable) (const FString & Name, bool & bFound) const<br> |
|  int64 | [**GetIntVariable**](#function-getintvariable) (const FString & Name, bool & bFound) const<br> |
|  TArray&lt; FString &gt; | [**GetStringSetVariable**](#function-getstringsetvariable) (const FString & Name, bool & bFound) const<br> |
|  FString | [**GetStringVariable**](#function-getstringvariable) (const FString & Name, bool & bFound) const<br> |
|  void | [**Initialize**](#function-initialize) (TWeakPtr&lt; [**FTryllAgent**](class_f_tryll_agent.md) &gt; InAgent) <br> |
|  bool | [**IsDirty**](#function-isdirty) () const<br> |
|  [**FTryllError**](struct_f_tryll_error.md) | [**RemoveFromSetVariable**](#function-removefromsetvariable) (const FString & Name, const FString & Element) <br> |
|  [**FTryllError**](struct_f_tryll_error.md) | [**ResetVariable**](#function-resetvariable) (const FString & Name) <br> |
|  [**FTryllError**](struct_f_tryll_error.md) | [**SetBoolVariable**](#function-setboolvariable) (const FString & Name, bool bValue) <br> |
|  [**FTryllError**](struct_f_tryll_error.md) | [**SetFloatVariable**](#function-setfloatvariable) (const FString & Name, float Value) <br> |
|  [**FTryllError**](struct_f_tryll_error.md) | [**SetIntVariable**](#function-setintvariable) (const FString & Name, int64 Value) <br> |
|  [**FTryllError**](struct_f_tryll_error.md) | [**SetStringSetVariable**](#function-setstringsetvariable) (const FString & Name, const TArray&lt; FString &gt; & Values) <br> |
|  [**FTryllError**](struct_f_tryll_error.md) | [**SetStringVariable**](#function-setstringvariable) (const FString & Name, const FString & Value) <br> |




























## Detailed Description


Blueprint-facing wrapper around [**FTryllAgentVariables**](class_f_tryll_agent_variables.md). Returned by [**UTryllAgentComponent::GetVariables()**](class_u_tryll_agent_component.md#function-getvariables); forwards typed setters/getters to the agent's C++ mirror. Reads are synchronous (served from the local mirror); writes are staged and flushed automatically before the next agent interaction. Setters return an [**FTryllError**](struct_f_tryll_error.md) (Ok on success); an unknown name or type mismatch is rejected locally with no wire round-trip. 


    
## Public Functions Documentation




### function AddToSetVariable 

```C++
FTryllError UTryllAgentVariables::AddToSetVariable (
    const FString & Name,
    const FString & Element
) 
```




<hr>



### function FlushIfDirty 

```C++
void UTryllAgentVariables::FlushIfDirty () 
```



Send every staged mutation as one batched UpdateAgentVariablesRequest. Normally automatic before Send/Resume/ChangeParams; use explicitly for debugging (Flush Now in Details). No-op success when nothing is staged. 


        

<hr>



### function GetBoolVariable 

```C++
bool UTryllAgentVariables::GetBoolVariable (
    const FString & Name,
    bool & bFound
) const
```




<hr>



### function GetFloatVariable 

```C++
float UTryllAgentVariables::GetFloatVariable (
    const FString & Name,
    bool & bFound
) const
```




<hr>



### function GetIntVariable 

```C++
int64 UTryllAgentVariables::GetIntVariable (
    const FString & Name,
    bool & bFound
) const
```




<hr>



### function GetStringSetVariable 

```C++
TArray< FString > UTryllAgentVariables::GetStringSetVariable (
    const FString & Name,
    bool & bFound
) const
```




<hr>



### function GetStringVariable 

```C++
FString UTryllAgentVariables::GetStringVariable (
    const FString & Name,
    bool & bFound
) const
```




<hr>



### function Initialize 

```C++
inline void UTryllAgentVariables::Initialize (
    TWeakPtr< FTryllAgent > InAgent
) 
```



Bind this wrapper to an agent. Safe to re-bind (e.g. after the agent is recreated). 


        

<hr>



### function IsDirty 

```C++
bool UTryllAgentVariables::IsDirty () const
```



True when one or more mutations are staged and not yet flushed to the server. 


        

<hr>



### function RemoveFromSetVariable 

```C++
FTryllError UTryllAgentVariables::RemoveFromSetVariable (
    const FString & Name,
    const FString & Element
) 
```




<hr>



### function ResetVariable 

```C++
FTryllError UTryllAgentVariables::ResetVariable (
    const FString & Name
) 
```




<hr>



### function SetBoolVariable 

```C++
FTryllError UTryllAgentVariables::SetBoolVariable (
    const FString & Name,
    bool bValue
) 
```




<hr>



### function SetFloatVariable 

```C++
FTryllError UTryllAgentVariables::SetFloatVariable (
    const FString & Name,
    float Value
) 
```




<hr>



### function SetIntVariable 

```C++
FTryllError UTryllAgentVariables::SetIntVariable (
    const FString & Name,
    int64 Value
) 
```




<hr>



### function SetStringSetVariable 

```C++
FTryllError UTryllAgentVariables::SetStringSetVariable (
    const FString & Name,
    const TArray< FString > & Values
) 
```




<hr>



### function SetStringVariable 

```C++
FTryllError UTryllAgentVariables::SetStringVariable (
    const FString & Name,
    const FString & Value
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllAgentVariables.h`

