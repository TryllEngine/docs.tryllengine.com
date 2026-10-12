

# File AgentVariables.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**cpp**](dir_bfb9b1426fa54ff893373dfbed66c670.md) **>** [**include**](dir_3428be636e548afd883ca44bef078c76.md) **>** [**tryll**](dir_3ee824def4798d48f3bfea3e4f65c1f8.md) **>** [**AgentVariables.h**](_agent_variables_8h.md)



_Client-side mirror for one agent's declared per-agent variables._ [More...](#detailed-description)

* `#include <tryll/TryllError.h>`
* `#include <chrono>`
* `#include <cstdint>`
* `#include <memory>`
* `#include <mutex>`
* `#include <optional>`
* `#include <string>`
* `#include <string_view>`
* `#include <unordered_map>`
* `#include <utility>`
* `#include <variant>`
* `#include <vector>`













## Namespaces

| Type | Name |
| ---: | :--- |
| namespace | [**Tryll**](namespace_tryll.md) <br> |
| namespace | [**Client**](namespace_tryll_1_1_client.md) <br> |
| namespace | [**Internal**](namespace_tryll_1_1_client_1_1_internal.md) <br> |


## Classes

| Type | Name |
| ---: | :--- |
| struct | [**AgentVariableDecl**](struct_tryll_1_1_client_1_1_agent_variable_decl.md) <br> |
| class | [**AgentVariables**](class_tryll_1_1_client_1_1_agent_variables.md) <br>_Client-side mirror + deferred-flush batcher for one agent's declared variables._  |
| struct | [**PendingUpdate**](struct_tryll_1_1_client_1_1_agent_variables_1_1_pending_update.md) <br> |


















































## Detailed Description


See docs/research/workflow/per-agent-variables-requirements.md for the full design. A variable's type is fixed by its initial value at [**Tryll::Client::TryllClient::CreateAgent**](class_tryll_1_1_client_1_1_tryll_client.md#function-createagent-12) — every write after that is validated locally against the declared type before anything reaches the wire. 


    

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/cpp/include/tryll/AgentVariables.h`

