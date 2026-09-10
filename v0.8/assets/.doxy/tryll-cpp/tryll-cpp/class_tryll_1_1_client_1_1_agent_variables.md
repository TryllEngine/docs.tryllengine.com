

# Class Tryll::Client::AgentVariables



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**AgentVariables**](class_tryll_1_1_client_1_1_agent_variables.md)



_Client-side mirror + deferred-flush batcher for one agent's declared variables._ [More...](#detailed-description)

* `#include <AgentVariables.h>`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**PendingUpdate**](struct_tryll_1_1_client_1_1_agent_variables_1_1_pending_update.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| enum std::uint8\_t | [**Op**](#enum-op)  <br> |




















## Public Functions

| Type | Name |
| ---: | :--- |
|  void | [**AddToSet**](#function-addtoset) (std::string\_view name, std::string\_view element) <br> |
|   | [**AgentVariables**](#function-agentvariables) () = default<br> |
|  std::optional&lt; [**VariableType**](namespace_tryll_1_1_client.md#enum-variabletype) &gt; | [**DeclaredType**](#function-declaredtype) (std::string\_view name) noexcept const<br> |
|  void | [**Flush**](#function-flush) () <br> |
|  std::optional&lt; bool &gt; | [**GetBool**](#function-getbool) (std::string\_view name) const<br> |
|  std::optional&lt; double &gt; | [**GetFloat**](#function-getfloat) (std::string\_view name) const<br> |
|  std::optional&lt; std::int64\_t &gt; | [**GetInt**](#function-getint) (std::string\_view name) const<br> |
|  std::optional&lt; std::string &gt; | [**GetString**](#function-getstring) (std::string\_view name) const<br> |
|  std::optional&lt; std::vector&lt; std::string &gt; &gt; | [**GetStringSet**](#function-getstringset) (std::string\_view name) const<br> |
|  void | [**HandleFlushResult**](#function-handleflushresult) (const std::optional&lt; [**TryllError**](class_tryll_1_1_client_1_1_tryll_error.md) &gt; & error) <br> |
|  bool | [**IsDeclared**](#function-isdeclared) (std::string\_view name) noexcept const<br> |
|  bool | [**IsDirty**](#function-isdirty) () noexcept const<br>_True if any write is staged and not yet sent to the server._  |
|  void | [**MergeServerValues**](#function-mergeservervalues) (const std::vector&lt; std::pair&lt; std::string, [**VariableValue**](namespace_tryll_1_1_client.md#typedef-variablevalue) &gt; &gt; & fetched) <br> |
|  void | [**RemoveFromSet**](#function-removefromset) (std::string\_view name, std::string\_view element) <br> |
|  void | [**Reset**](#function-reset) (std::string\_view name) <br> |
|  void | [**SetBool**](#function-setbool) (std::string\_view name, bool value) <br> |
|  void | [**SetFloat**](#function-setfloat) (std::string\_view name, double value) <br> |
|  void | [**SetInt**](#function-setint) (std::string\_view name, std::int64\_t value) <br> |
|  void | [**SetString**](#function-setstring) (std::string\_view name, std::string value) <br> |
|  void | [**SetStringSet**](#function-setstringset) (std::string\_view name, std::vector&lt; std::string &gt; value) <br> |
|  std::vector&lt; std::pair&lt; std::string, [**VariableValue**](namespace_tryll_1_1_client.md#typedef-variablevalue) &gt; &gt; | [**Snapshot**](#function-snapshot) () const<br> |
|  void | [**SyncFromServer**](#function-syncfromserver) (std::vector&lt; std::string &gt; names={}, std::chrono::milliseconds timeout=std::chrono::seconds{10}) <br> |




























## Detailed Description


Obtained via [**Tryll::Client::AgentProxy::Variables**](class_tryll_1_1_client_1_1_agent_proxy.md#function-variables). Reads are synchronous, served from the local mirror — only the client writes variables in v1, so no server round-trip is needed to read one back. Writes are validated locally (unknown name / type mismatch throws [**Tryll::Client::TryllError**](class_tryll_1_1_client_1_1_tryll_error.md) immediately, no wire traffic) and staged; the proxy flushes any staged writes as one batched `UpdateAgentVariablesRequest` automatically, immediately before the next agent interaction (`SendText` / `Resume` / `ChangeParams` — not `Cancel`). Staged writes are coalesced per name (last value wins for scalars; a minimal add/remove delta, or a whole-set assign, whichever is smaller, for sets) so rapid repeated writes never grow the batch unboundedly.


Flushed writes are non-blocking. The batch is retained in-flight until the server acknowledges it: on a transient `AgentBusy` the batch is re-staged behind any newer writes and retried on the next interaction; on a hard rejection (rare, since the mirror pre-validates unknown-name/type-mismatch locally) the affected names are re-synced from the server so the mirror never stays silently diverged. Either way the error surfaces through the agent's usual [**Tryll::Client::AgentProxy::SetOnError**](class_tryll_1_1_client_1_1_agent_proxy.md#function-setonerror) callback, the same path `SendText` errors use. 


    
## Public Types Documentation




### enum Op 

```C++
enum Tryll::Client::AgentVariables::Op {
    Assign,
    AddToSet,
    RemoveFromSet,
    Reset
};
```



One staged mutation, in wire-op terms. Exposed (rather than kept private) only so TryllClientImpl can encode it — treat as implementation detail, not a stable public API. 


        

<hr>
## Public Functions Documentation




### function AddToSet 

```C++
void Tryll::Client::AgentVariables::AddToSet (
    std::string_view name,
    std::string_view element
) 
```



Idempotent — no-op if `element` is already present. 

**Exception:**


* `TryllError` If `name` is not declared or is not a StringSet. 




        

<hr>



### function AgentVariables 

```C++
Tryll::Client::AgentVariables::AgentVariables () = default
```




<hr>



### function DeclaredType 

```C++
std::optional< VariableType > Tryll::Client::AgentVariables::DeclaredType (
    std::string_view name
) noexcept const
```




<hr>



### function Flush 

```C++
void Tryll::Client::AgentVariables::Flush () 
```



Send any staged writes now as one batched request, instead of waiting for the next SendText/Resume/ChangeParams call. Fire-and-forget, same as the automatic flush — errors surface via the agent's OnError callback. No-op if nothing is staged. 


        

<hr>



### function GetBool 

```C++
std::optional< bool > Tryll::Client::AgentVariables::GetBool (
    std::string_view name
) const
```




<hr>



### function GetFloat 

```C++
std::optional< double > Tryll::Client::AgentVariables::GetFloat (
    std::string_view name
) const
```




<hr>



### function GetInt 

```C++
std::optional< std::int64_t > Tryll::Client::AgentVariables::GetInt (
    std::string_view name
) const
```




<hr>



### function GetString 

```C++
std::optional< std::string > Tryll::Client::AgentVariables::GetString (
    std::string_view name
) const
```




<hr>



### function GetStringSet 

```C++
std::optional< std::vector< std::string > > Tryll::Client::AgentVariables::GetStringSet (
    std::string_view name
) const
```




<hr>



### function HandleFlushResult 

```C++
void Tryll::Client::AgentVariables::HandleFlushResult (
    const std::optional< TryllError > & error
) 
```



Result of a flushed batch, routed back from the reader thread via TryllClientImpl::OnVariableFlushResult. `error` is nullopt on success. Exposed only so TryllClientImpl can invoke it; treat as internal. 


        

<hr>



### function IsDeclared 

```C++
bool Tryll::Client::AgentVariables::IsDeclared (
    std::string_view name
) noexcept const
```




<hr>



### function IsDirty 

_True if any write is staged and not yet sent to the server._ 
```C++
bool Tryll::Client::AgentVariables::IsDirty () noexcept const
```




<hr>



### function MergeServerValues 

```C++
void Tryll::Client::AgentVariables::MergeServerValues (
    const std::vector< std::pair< std::string, VariableValue > > & fetched
) 
```



Merge server-fetched values into the mirror, skipping any name with a locally newer (staged or in-flight) write. Exposed only for TryllClientImpl's async re-sync path; treat as internal. 


        

<hr>



### function RemoveFromSet 

```C++
void Tryll::Client::AgentVariables::RemoveFromSet (
    std::string_view name,
    std::string_view element
) 
```



Idempotent — no-op if `element` is already absent. 

**Exception:**


* `TryllError` If `name` is not declared or is not a StringSet. 




        

<hr>



### function Reset 

```C++
void Tryll::Client::AgentVariables::Reset (
    std::string_view name
) 
```



Restores the CreateAgent initial value. 

**Exception:**


* `TryllError` If `name` is not declared. 




        

<hr>



### function SetBool 

```C++
void Tryll::Client::AgentVariables::SetBool (
    std::string_view name,
    bool value
) 
```





**Exception:**


* `TryllError` If `name` is not declared, or its declared type is not Bool. 




        

<hr>



### function SetFloat 

```C++
void Tryll::Client::AgentVariables::SetFloat (
    std::string_view name,
    double value
) 
```





**Exception:**


* `TryllError` If `name` is not declared, or its declared type is not Float. 




        

<hr>



### function SetInt 

```C++
void Tryll::Client::AgentVariables::SetInt (
    std::string_view name,
    std::int64_t value
) 
```





**Exception:**


* `TryllError` If `name` is not declared, or its declared type is not Int. 




        

<hr>



### function SetString 

```C++
void Tryll::Client::AgentVariables::SetString (
    std::string_view name,
    std::string value
) 
```





**Exception:**


* `TryllError` If `name` is not declared, or its declared type is not String. 




        

<hr>



### function SetStringSet 

```C++
void Tryll::Client::AgentVariables::SetStringSet (
    std::string_view name,
    std::vector< std::string > value
) 
```





**Exception:**


* `TryllError` If `name` is not declared, or its declared type is not StringSet. 




        

<hr>



### function Snapshot 

```C++
std::vector< std::pair< std::string, VariableValue > > Tryll::Client::AgentVariables::Snapshot () const
```



Every declared variable and its current mirrored value, in no particular order. Mainly for diagnostics/debug UI (e.g. test-chat's `/var show`). 


        

<hr>



### function SyncFromServer 

```C++
void Tryll::Client::AgentVariables::SyncFromServer (
    std::vector< std::string > names={},
    std::chrono::milliseconds timeout=std::chrono::seconds{10}
) 
```



Debug/verification round-trip: fetch current values from the server and merge them into the local mirror (does not touch pending/staged writes). Normal reads never need this — only the client writes variables in v1, so the mirror is authoritative; use this only to double-check server state. `names` empty means every declared variable. Blocks until the response arrives or `timeout` elapses. 

**Exception:**


* `TryllError` On server error, disconnect, or timeout. 




        

<hr>## Friends Documentation





### friend TryllClientImpl 

```C++
class Tryll::Client::AgentVariables::TryllClientImpl (
    Internal::TryllClientImpl
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/AgentVariables.h`

