

# Struct Tryll::Client::AgentProxy::IntentClassifiedEvent



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**AgentProxy**](class_tryll_1_1_client_1_1_agent_proxy.md) **>** [**IntentClassifiedEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_intent_classified_event.md)



_Owning payload for_ `event_type="intent_classified"` _(embedding classifier)._

* `#include <AgentProxy.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  float | [**distance**](#variable-distance)   = `0.f`<br> |
|  std::string | [**intent**](#variable-intent)  <br> |
|  std::string | [**nodeName**](#variable-nodename)  <br> |
|  std::string | [**notFoundReason**](#variable-notfoundreason)  <br> |
|  std::string | [**recordId**](#variable-recordid)  <br> |
|  std::size\_t | [**recordIndex**](#variable-recordindex)   = `0`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  bool | [**Accepted**](#function-accepted) () noexcept const<br> |




























## Public Attributes Documentation




### variable distance 

```C++
float Tryll::Client::AgentProxy::IntentClassifiedEvent::distance;
```




<hr>



### variable intent 

```C++
std::string Tryll::Client::AgentProxy::IntentClassifiedEvent::intent;
```




<hr>



### variable nodeName 

```C++
std::string Tryll::Client::AgentProxy::IntentClassifiedEvent::nodeName;
```




<hr>



### variable notFoundReason 

```C++
std::string Tryll::Client::AgentProxy::IntentClassifiedEvent::notFoundReason;
```




<hr>



### variable recordId 

```C++
std::string Tryll::Client::AgentProxy::IntentClassifiedEvent::recordId;
```




<hr>



### variable recordIndex 

```C++
std::size_t Tryll::Client::AgentProxy::IntentClassifiedEvent::recordIndex;
```




<hr>
## Public Functions Documentation




### function Accepted 

```C++
inline bool Tryll::Client::AgentProxy::IntentClassifiedEvent::Accepted () noexcept const
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/AgentProxy.h`

