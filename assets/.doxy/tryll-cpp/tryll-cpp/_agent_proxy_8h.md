

# File AgentProxy.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**cpp**](dir_bfb9b1426fa54ff893373dfbed66c670.md) **>** [**include**](dir_3428be636e548afd883ca44bef078c76.md) **>** [**tryll**](dir_3ee824def4798d48f3bfea3e4f65c1f8.md) **>** [**AgentProxy.h**](_agent_proxy_8h.md)



_Client-side handle for a server-hosted Tryll agent._ [More...](#detailed-description)

* `#include <tryll/AgentKvCache.h>`
* `#include <tryll/AgentVariables.h>`
* `#include <tryll/DialogSnapshot.h>`
* `#include <tryll/GraphDescription.h>`
* `#include <tryll/TryllError.h>`
* `#include <messages_generated.h>`
* `#include <cstddef>`
* `#include <cstdint>`
* `#include <functional>`
* `#include <future>`
* `#include <memory>`
* `#include <span>`
* `#include <string>`
* `#include <string_view>`
* `#include <utility>`
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
| class | [**AgentProxy**](class_tryll_1_1_client_1_1_agent_proxy.md) <br>_Client-side handle for one server-side agent._  |
| struct | [**DialogInteraction**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_dialog_interaction.md) <br> |
| struct | [**IntentClassifiedEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_intent_classified_event.md) <br>_Owning payload for_ `event_type="intent_classified"` _(embedding classifier)._ |
| struct | [**IntentLlmClassifiedEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_intent_llm_classified_event.md) <br>_Owning payload for_ `event_type="intent_llm_classified"` _._ |
| struct | [**IntentLlmLabelProb**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_intent_llm_label_prob.md) <br>_One label / first-token probability pair from ClassifyIntentLLM._  |
| struct | [**ToolCallEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_tool_call_event.md) <br>_Owning tool-call event for deferred / async handling._  |
| struct | [**ToolResult**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_tool_result.md) <br>_One model-visible result correlated to a tool\_call event's call\_id._  |


















































## Detailed Description



------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/cpp/include/tryll/AgentProxy.h`

