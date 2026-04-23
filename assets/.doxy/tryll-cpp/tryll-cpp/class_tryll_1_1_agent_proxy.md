

# Class Tryll::AgentProxy



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**AgentProxy**](class_tryll_1_1_agent_proxy.md)



_Client-side handle for one server-side agent._ [More...](#detailed-description)

* `#include <AgentProxy.h>`

















## Public Types

| Type | Name |
| ---: | :--- |
| typedef std::function&lt; void(std::string\_view toolName, std::string\_view argumentsJson)&gt; | [**ToolCallCallback**](#typedef-toolcallcallback)  <br>_Callback type for tool-call notifications._  |




















## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**AgentProxy**](#function-agentproxy-12) () = default<br> |
|  void | [**ChangeParam**](#function-changeparam) (std::string\_view nodeName, std::string\_view paramName, std::string\_view paramValue) <br>_Blocking convenience wrapper over_ [_**ChangeParamAsync**_](class_tryll_1_1_agent_proxy.md#function-changeparamasync) _._ |
|  std::future&lt; void &gt; | [**ChangeParamAsync**](#function-changeparamasync) (std::string\_view nodeName, std::string\_view paramName, std::string\_view paramValue) <br>_Mutate a named parameter on a workflow node asynchronously._  |
|  void | [**Destroy**](#function-destroy) () <br>_Blocking convenience wrapper over_ [_**DestroyAsync**_](class_tryll_1_1_agent_proxy.md#function-destroyasync) _._ |
|  std::future&lt; void &gt; | [**DestroyAsync**](#function-destroyasync) () <br>_Request agent destruction on the server asynchronously._  |
|  std::uint64\_t | [**GetAgentId**](#function-getagentid) () noexcept const<br>_Server-assigned agent identifier._  |
|  void | [**SendText**](#function-sendtext) (std::string\_view text, std::function&lt; void(std::string\_view text, bool isDelta, bool isFinal)&gt; onAnswerText={}) <br>_Blocking convenience wrapper over_ [_**SendTextAsync**_](class_tryll_1_1_agent_proxy.md#function-sendtextasync) _._ |
|  std::future&lt; void &gt; | [**SendTextAsync**](#function-sendtextasync) (std::string\_view text, std::function&lt; void(std::string\_view text, bool isDelta, bool isFinal)&gt; onAnswerText={}) <br>_Send a text message to the agent asynchronously._  |
|  void | [**SetOnToolCall**](#function-setontoolcall) ([**ToolCallCallback**](class_tryll_1_1_agent_proxy.md#typedef-toolcallcallback) cb) <br>_Register a callback to be invoked on each tool-call notification._  |




























## Detailed Description


Obtained from [**TryllClient::CreateAgent**](class_tryll_1_1_tryll_client.md#function-createagent). The proxy is copyable — all copies share the same underlying agent binding; the server-side agent is destroyed when [**Destroy**](class_tryll_1_1_agent_proxy.md#function-destroy) is called explicitly. 


    
## Public Types Documentation




### typedef ToolCallCallback 

_Callback type for tool-call notifications._ 
```C++
using Tryll::AgentProxy::ToolCallCallback = 
std::function<void(std::string_view toolName, std::string_view argumentsJson)>;
```



Invoked on the reader thread for each `ToolCallNotification` received from the server (i.e. when the agent's graph has a `ToolCall` node with `notify_client` = "true").


Parameters:
* `toolName` — the tool the model asked to call.
* `argumentsJson` — compact JSON object of arguments, e.g. `{"city"`:"Berlin"}.




Keep the callback short and non-blocking (same contract as the `onAnswerText` streaming callback on [**SendTextAsync**](class_tryll_1_1_agent_proxy.md#function-sendtextasync)). 


        

<hr>
## Public Functions Documentation




### function AgentProxy [1/2]

```C++
Tryll::AgentProxy::AgentProxy () = default
```




<hr>



### function ChangeParam 

_Blocking convenience wrapper over_ [_**ChangeParamAsync**_](class_tryll_1_1_agent_proxy.md#function-changeparamasync) _._
```C++
void Tryll::AgentProxy::ChangeParam (
    std::string_view nodeName,
    std::string_view paramName,
    std::string_view paramValue
) 
```





**Exception:**


* [**TryllError**](class_tryll_1_1_tryll_error.md) On server-reported errors or disconnect. 




        

<hr>



### function ChangeParamAsync 

_Mutate a named parameter on a workflow node asynchronously._ 
```C++
std::future< void > Tryll::AgentProxy::ChangeParamAsync (
    std::string_view nodeName,
    std::string_view paramName,
    std::string_view paramValue
) 
```



The agent must not be processing a turn; if it is, the returned future completes with a [**TryllError**](class_tryll_1_1_tryll_error.md) carrying error code `AgentBusy` (3004).




**Parameters:**


* `nodeName` Instance name of the target node (as declared in the graph). 
* `paramName` Parameter key to set. 
* `paramValue` New value as a string.



**Returns:**

Future completing on `Ack`. Throws [**TryllError**](class_tryll_1_1_tryll_error.md) on any server-reported error (UnknownNode 3005, ParamNotMutable 3006, InvalidParamValue 3007, or AgentBusy 3004). 





        

<hr>



### function Destroy 

_Blocking convenience wrapper over_ [_**DestroyAsync**_](class_tryll_1_1_agent_proxy.md#function-destroyasync) _._
```C++
void Tryll::AgentProxy::Destroy () 
```





**Exception:**


* [**TryllError**](class_tryll_1_1_tryll_error.md) On server-reported errors, disconnect, or timeout (configured by the owning client). 




        

<hr>



### function DestroyAsync 

_Request agent destruction on the server asynchronously._ 
```C++
std::future< void > Tryll::AgentProxy::DestroyAsync () 
```



The proxy becomes inert once the returned future completes; further `SendText` calls fail with [**TryllError**](class_tryll_1_1_tryll_error.md).




**Returns:**

Future completing when the server acknowledges destruction. 





        

<hr>



### function GetAgentId 

_Server-assigned agent identifier._ 
```C++
inline std::uint64_t Tryll::AgentProxy::GetAgentId () noexcept const
```





**Returns:**

The `agent_id` field from the original `CreateAgentResponse`. 





        

<hr>



### function SendText 

_Blocking convenience wrapper over_ [_**SendTextAsync**_](class_tryll_1_1_agent_proxy.md#function-sendtextasync) _._
```C++
void Tryll::AgentProxy::SendText (
    std::string_view text,
    std::function< void(std::string_view text, bool isDelta, bool isFinal)> onAnswerText={}
) 
```





**Parameters:**


* `text` User-turn text to send. 
* `onAnswerText` Optional streaming callback; see [**SendTextAsync**](class_tryll_1_1_agent_proxy.md#function-sendtextasync).



**Exception:**


* [**TryllError**](class_tryll_1_1_tryll_error.md) On server-reported errors or disconnect. 




        

<hr>



### function SendTextAsync 

_Send a text message to the agent asynchronously._ 
```C++
std::future< void > Tryll::AgentProxy::SendTextAsync (
    std::string_view text,
    std::function< void(std::string_view text, bool isDelta, bool isFinal)> onAnswerText={}
) 
```



The returned future completes (with `void`) when `TurnComplete` arrives. Any streaming `AnswerText` frames received before `TurnComplete` are dispatched to `onAnswerText` on the reader thread; keep the callback short and non-blocking.




**Parameters:**


* `text` User-turn text to send. 
* `onAnswerText` Optional streaming callback invoked with (chunk, isDelta, isFinal) for each streamed chunk. Ignored when empty.



**Returns:**

Future completing when the turn finishes. Calling `future::get()` propagates any [**TryllError**](class_tryll_1_1_tryll_error.md) reported by the server or raised locally on disconnect.




**Exception:**


* [**TryllError**](class_tryll_1_1_tryll_error.md) Only via the returned future; never directly. 




        

<hr>



### function SetOnToolCall 

_Register a callback to be invoked on each tool-call notification._ 
```C++
void Tryll::AgentProxy::SetOnToolCall (
    ToolCallCallback cb
) 
```



Replaces any previously set callback. Pass an empty (default- constructed) [**ToolCallCallback**](class_tryll_1_1_agent_proxy.md#typedef-toolcallcallback) to unregister.


The callback fires on the reader thread for every `ToolCallNotification` frame the server sends for this agent. It must return quickly and must not call any blocking [**TryllClient**](class_tryll_1_1_tryll_client.md) or [**AgentProxy**](class_tryll_1_1_agent_proxy.md) methods.




**Parameters:**


* `cb` Callable to invoke with ``(toolName, argumentsJson), or empty to clear the current callback. 




        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/tryll-mono/server/client-cpp/include/tryll/AgentProxy.h`

