

# Class FTryllAgentVariables



[**ClassList**](annotated.md) **>** [**FTryllAgentVariables**](class_f_tryll_agent_variables.md)



[More...](#detailed-description)

* `#include <TryllAgentVariables.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  TFunction&lt; void(TArray&lt; [**FTryllVariableUpdate**](struct_f_tryll_variable_update.md) &gt;, TFunction&lt; void(const [**FTryllError**](struct_f_tryll_error.md) &)&gt;)&gt; | [**TestSendUpdate**](#variable-testsendupdate)  <br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  [**FTryllError**](struct_f_tryll_error.md) | [**AddToSet**](#function-addtoset) (const FString & Name, const FString & Element) <br> |
|  TOptional&lt; ETryllVariableType &gt; | [**DeclaredType**](#function-declaredtype) (const FString & Name) const<br> |
|   | [**FTryllAgentVariables**](#function-ftryllagentvariables) ([**FTryllAgent**](class_f_tryll_agent.md) \* InAgent, const TArray&lt; [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) &gt; & Declared) <br> |
|  void | [**FlushIfDirty**](#function-flushifdirty) (TFunction&lt; void(const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete=nullptr, uint64 CarrierRequestId=0) <br> |
|  bool | [**GetBool**](#function-getbool) (const FString & Name) const<br> |
|  double | [**GetFloat**](#function-getfloat) (const FString & Name) const<br> |
|  int64 | [**GetInt**](#function-getint) (const FString & Name) const<br> |
|  FString | [**GetString**](#function-getstring) (const FString & Name) const<br> |
|  TArray&lt; FString &gt; | [**GetStringSet**](#function-getstringset) (const FString & Name) const<br> |
|  bool | [**IsDeclared**](#function-isdeclared) (const FString & Name) const<br> |
|  bool | [**IsDirty**](#function-isdirty) () const<br> |
|  [**FTryllError**](struct_f_tryll_error.md) | [**RemoveFromSet**](#function-removefromset) (const FString & Name, const FString & Element) <br> |
|  [**FTryllError**](struct_f_tryll_error.md) | [**Reset**](#function-reset) (const FString & Name) <br> |
|  [**FTryllError**](struct_f_tryll_error.md) | [**SetBool**](#function-setbool) (const FString & Name, bool bValue) <br> |
|  [**FTryllError**](struct_f_tryll_error.md) | [**SetFloat**](#function-setfloat) (const FString & Name, double Value) <br> |
|  [**FTryllError**](struct_f_tryll_error.md) | [**SetInt**](#function-setint) (const FString & Name, int64 Value) <br> |
|  [**FTryllError**](struct_f_tryll_error.md) | [**SetString**](#function-setstring) (const FString & Name, const FString & Value) <br> |
|  [**FTryllError**](struct_f_tryll_error.md) | [**SetStringSet**](#function-setstringset) (const FString & Name, const TArray&lt; FString &gt; & Values) <br> |
|  TMap&lt; FString, [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) &gt; | [**Snapshot**](#function-snapshot) () const<br> |
|  void | [**SyncFromServerAsync**](#function-syncfromserverasync) (TArray&lt; FString &gt; Names={}, TFunction&lt; void(const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete=nullptr) <br> |




























## Detailed Description


Local mirror of one agent's declared per-agent Variables (C++ surface). Obtained via [**FTryllAgent::GetVariables()**](class_f_tryll_agent.md#function-getvariables). Blueprint code uses the [**UTryllAgentVariables**](class_u_tryll_agent_variables.md) wrapper ([**UTryllAgentComponent::GetVariables()**](class_u_tryll_agent_component.md#function-getvariables)) instead. 


    
## Public Attributes Documentation




### variable TestSendUpdate 

```C++
TFunction<void(TArray<FTryllVariableUpdate>, TFunction<void(const FTryllError&)>)> FTryllAgentVariables::TestSendUpdate;
```



Test hook: when set, FlushIfDirty invokes this instead of the live agent wire send. Signature is (WireBatch, OnComplete). 


        

<hr>
## Public Functions Documentation




### function AddToSet 

```C++
FTryllError FTryllAgentVariables::AddToSet (
    const FString & Name,
    const FString & Element
) 
```



Idempotent — a no-op success if Element is already present. 


        

<hr>



### function DeclaredType 

```C++
TOptional< ETryllVariableType > FTryllAgentVariables::DeclaredType (
    const FString & Name
) const
```




<hr>



### function FTryllAgentVariables 

```C++
FTryllAgentVariables::FTryllAgentVariables (
    FTryllAgent * InAgent,
    const TArray< FTryllVariableDecl > & Declared
) 
```




<hr>



### function FlushIfDirty 

```C++
void FTryllAgentVariables::FlushIfDirty (
    TFunction< void(const FTryllError &)> OnComplete=nullptr,
    uint64 CarrierRequestId=0
) 
```



Send every staged mutation as one batched UpdateAgentVariablesRequest. Called automatically at the top of [**FTryllAgent::SendMessage**](class_f_tryll_agent.md#function-sendmessage)/Resume/ ChangeParams — deliberately NOT before Cancel. No-op (invokes OnComplete with Ok) when nothing is staged. 


        

<hr>



### function GetBool 

```C++
bool FTryllAgentVariables::GetBool (
    const FString & Name
) const
```




<hr>



### function GetFloat 

```C++
double FTryllAgentVariables::GetFloat (
    const FString & Name
) const
```




<hr>



### function GetInt 

```C++
int64 FTryllAgentVariables::GetInt (
    const FString & Name
) const
```




<hr>



### function GetString 

```C++
FString FTryllAgentVariables::GetString (
    const FString & Name
) const
```




<hr>



### function GetStringSet 

```C++
TArray< FString > FTryllAgentVariables::GetStringSet (
    const FString & Name
) const
```




<hr>



### function IsDeclared 

```C++
inline bool FTryllAgentVariables::IsDeclared (
    const FString & Name
) const
```




<hr>



### function IsDirty 

```C++
inline bool FTryllAgentVariables::IsDirty () const
```



True when one or more mutations are staged and not yet flushed. 


        

<hr>



### function RemoveFromSet 

```C++
FTryllError FTryllAgentVariables::RemoveFromSet (
    const FString & Name,
    const FString & Element
) 
```



Idempotent — a no-op success if Element is already absent. 


        

<hr>



### function Reset 

```C++
FTryllError FTryllAgentVariables::Reset (
    const FString & Name
) 
```



Restores the CreateAgent-time initial value for Name. 


        

<hr>



### function SetBool 

```C++
FTryllError FTryllAgentVariables::SetBool (
    const FString & Name,
    bool bValue
) 
```




<hr>



### function SetFloat 

```C++
FTryllError FTryllAgentVariables::SetFloat (
    const FString & Name,
    double Value
) 
```




<hr>



### function SetInt 

```C++
FTryllError FTryllAgentVariables::SetInt (
    const FString & Name,
    int64 Value
) 
```




<hr>



### function SetString 

```C++
FTryllError FTryllAgentVariables::SetString (
    const FString & Name,
    const FString & Value
) 
```




<hr>



### function SetStringSet 

```C++
FTryllError FTryllAgentVariables::SetStringSet (
    const FString & Name,
    const TArray< FString > & Values
) 
```




<hr>



### function Snapshot 

```C++
inline TMap< FString, FTryllVariableDecl > FTryllAgentVariables::Snapshot () const
```



Copy of every declared variable's current local value, keyed by name. 


        

<hr>



### function SyncFromServerAsync 

```C++
void FTryllAgentVariables::SyncFromServerAsync (
    TArray< FString > Names={},
    TFunction< void(const FTryllError &)> OnComplete=nullptr
) 
```



Refresh the local mirror from the server's authoritative current values. Names with a locally newer (staged or in-flight) write are skipped. Empty Names = every declared variable. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllAgentVariables.h`

