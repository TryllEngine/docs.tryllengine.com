

# File AgentProxy.h



[**FileList**](files.md) **>** [**client-cpp**](dir_609af48e6c91725f4b3bc7ea6c8d260d.md) **>** [**include**](dir_e007c6614a1ec85f1d1448223783a2c8.md) **>** [**tryll**](dir_8463b878364fc27e5e1890816f6531c6.md) **>** [**AgentProxy.h**](_agent_proxy_8h.md)



_Client-side handle for a server-hosted Tryll agent._ [More...](#detailed-description)

* `#include <tryll/TryllError.h>`
* `#include <cstdint>`
* `#include <functional>`
* `#include <future>`
* `#include <memory>`
* `#include <string_view>`













## Namespaces

| Type | Name |
| ---: | :--- |
| namespace | [**Tryll**](namespace_tryll.md) <br> |


## Classes

| Type | Name |
| ---: | :--- |
| class | [**AgentProxy**](class_tryll_1_1_agent_proxy.md) <br>_Client-side handle for one server-side agent._  |


















































## Detailed Description


An [**Tryll::AgentProxy**](class_tryll_1_1_agent_proxy.md) is returned by [**Tryll::TryllClient::CreateAgent**](class_tryll_1_1_tryll_client.md#function-createagent) (or its async twin) and represents one agent running a workflow graph on the server. All turn-driving APIs live on the proxy; the owning [**Tryll::TryllClient**](class_tryll_1_1_tryll_client.md) handles the underlying session. 


    

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo/server/client-cpp/include/tryll/AgentProxy.h`

