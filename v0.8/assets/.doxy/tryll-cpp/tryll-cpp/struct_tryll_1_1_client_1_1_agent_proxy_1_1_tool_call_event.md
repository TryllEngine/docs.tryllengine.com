

# Struct Tryll::Client::AgentProxy::ToolCallEvent



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**AgentProxy**](class_tryll_1_1_client_1_1_agent_proxy.md) **>** [**ToolCallEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_tool_call_event.md)



_Owning tool-call event for deferred / async handling._ [More...](#detailed-description)

* `#include <AgentProxy.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::string | [**argumentsJson**](#variable-argumentsjson)  <br> |
|  std::string | [**callId**](#variable-callid)  <br> |
|  std::string | [**disposition**](#variable-disposition)  <br> |
|  std::string | [**nodeName**](#variable-nodename)  <br> |
|  std::string | [**toolName**](#variable-toolname)  <br> |












































## Detailed Description


Unlike [**ToolCallCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-toolcallcallback) / [**ToolCallWithIdCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-toolcallwithidcallback), all fields are owned `std::string` values safe to capture beyond the reader-thread dispatch. Prefer [**SetOnToolCallEvent**](class_tryll_1_1_client_1_1_agent_proxy.md#function-setontoolcallevent) for new code. 


    
## Public Attributes Documentation




### variable argumentsJson 

```C++
std::string Tryll::Client::AgentProxy::ToolCallEvent::argumentsJson;
```




<hr>



### variable callId 

```C++
std::string Tryll::Client::AgentProxy::ToolCallEvent::callId;
```




<hr>



### variable disposition 

```C++
std::string Tryll::Client::AgentProxy::ToolCallEvent::disposition;
```




<hr>



### variable nodeName 

```C++
std::string Tryll::Client::AgentProxy::ToolCallEvent::nodeName;
```




<hr>



### variable toolName 

```C++
std::string Tryll::Client::AgentProxy::ToolCallEvent::toolName;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/AgentProxy.h`

