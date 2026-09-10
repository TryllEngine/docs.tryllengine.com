

# Struct Tryll::Client::AgentProxy::DialogInteraction



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**AgentProxy**](class_tryll_1_1_client_1_1_agent_proxy.md) **>** [**DialogInteraction**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_dialog_interaction.md)



[More...](#detailed-description)

* `#include <AgentProxy.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::string | [**assistantMessage**](#variable-assistantmessage)  <br> |
|  std::string | [**userMessage**](#variable-usermessage)  <br> |












































## Detailed Description


One scripted exchange for [**AppendInteractionsAsync**](class_tryll_1_1_client_1_1_agent_proxy.md#function-appendinteractionsasync). Empty strings omit that side; both empty → the pair is skipped server-side. 


    
## Public Attributes Documentation




### variable assistantMessage 

```C++
std::string Tryll::Client::AgentProxy::DialogInteraction::assistantMessage;
```




<hr>



### variable userMessage 

```C++
std::string Tryll::Client::AgentProxy::DialogInteraction::userMessage;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/AgentProxy.h`

