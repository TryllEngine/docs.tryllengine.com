

# Struct Tryll::Client::AgentProxy::IntentLlmClassifiedEvent



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**AgentProxy**](class_tryll_1_1_client_1_1_agent_proxy.md) **>** [**IntentLlmClassifiedEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_intent_llm_classified_event.md)



_Owning payload for_ `event_type="intent_llm_classified"` _._

* `#include <AgentProxy.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::string | [**intent**](#variable-intent)  <br> |
|  std::vector&lt; [**IntentLlmLabelProb**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_intent_llm_label_prob.md) &gt; | [**labelProbs**](#variable-labelprobs)  <br> |
|  std::string | [**nodeName**](#variable-nodename)  <br> |
|  std::string | [**notFoundReason**](#variable-notfoundreason)  <br> |
|  float | [**secondProb**](#variable-secondprob)   = `0.f`<br> |
|  float | [**topProb**](#variable-topprob)   = `0.f`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  bool | [**Accepted**](#function-accepted) () noexcept const<br> |




























## Public Attributes Documentation




### variable intent 

```C++
std::string Tryll::Client::AgentProxy::IntentLlmClassifiedEvent::intent;
```




<hr>



### variable labelProbs 

```C++
std::vector<IntentLlmLabelProb> Tryll::Client::AgentProxy::IntentLlmClassifiedEvent::labelProbs;
```




<hr>



### variable nodeName 

```C++
std::string Tryll::Client::AgentProxy::IntentLlmClassifiedEvent::nodeName;
```




<hr>



### variable notFoundReason 

```C++
std::string Tryll::Client::AgentProxy::IntentLlmClassifiedEvent::notFoundReason;
```




<hr>



### variable secondProb 

```C++
float Tryll::Client::AgentProxy::IntentLlmClassifiedEvent::secondProb;
```




<hr>



### variable topProb 

```C++
float Tryll::Client::AgentProxy::IntentLlmClassifiedEvent::topProb;
```




<hr>
## Public Functions Documentation




### function Accepted 

```C++
inline bool Tryll::Client::AgentProxy::IntentLlmClassifiedEvent::Accepted () noexcept const
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/AgentProxy.h`

