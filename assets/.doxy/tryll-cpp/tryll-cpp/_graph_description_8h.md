

# File GraphDescription.h



[**FileList**](files.md) **>** [**client-cpp**](dir_609af48e6c91725f4b3bc7ea6c8d260d.md) **>** [**include**](dir_e007c6614a1ec85f1d1448223783a2c8.md) **>** [**tryll**](dir_8463b878364fc27e5e1890816f6531c6.md) **>** [**GraphDescription.h**](_graph_description_8h.md)



_Workflow graph builder and associated value types for the C++ client._ [More...](#detailed-description)

* `#include <cstdint>`
* `#include <optional>`
* `#include <string>`
* `#include <unordered_map>`
* `#include <vector>`













## Namespaces

| Type | Name |
| ---: | :--- |
| namespace | [**Tryll**](namespace_tryll.md) <br> |
| namespace | [**Client**](namespace_tryll_1_1_client.md) <br> |


## Classes

| Type | Name |
| ---: | :--- |
| class | [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) <br>_Fluent builder for a workflow graph description sent to the server._  |
| struct | [**NodeDesc**](struct_tryll_1_1_client_1_1_graph_description_1_1_node_desc.md) <br>_Internal serialisable description of one node._  |
| struct | [**RouteDesc**](struct_tryll_1_1_client_1_1_graph_description_1_1_route_desc.md) <br>_Internal serialisable description of one wire between two nodes._  |
| struct | [**KnowledgePresentationConfig**](struct_tryll_1_1_client_1_1_knowledge_presentation_config.md) <br>_Per-agent knowledge presentation configuration._  |
| struct | [**RetrievePresentationConfig**](struct_tryll_1_1_client_1_1_retrieve_presentation_config.md) <br>_Formatting rules for one knowledge source (or the default for all sources)._  |
| struct | [**ToolDef**](struct_tryll_1_1_client_1_1_tool_def.md) <br>_Definition of a callable tool for a ToolCall node._  |
| struct | [**ToolParamDef**](struct_tryll_1_1_client_1_1_tool_param_def.md) <br>_Description of one parameter of a tool definition._  |


















































## Detailed Description


Declares [**Tryll::Client::GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) (the fluent builder used on [**Tryll::TryllClient::CreateAgent**](class_tryll_1_1_tryll_client.md#function-createagent)) along with the enums and structs referenced by the graph — [**Tryll::Client::NodeType**](namespace_tryll_1_1_client.md#enum-nodetype), [**Tryll::Client::InferenceEngine**](namespace_tryll_1_1_client.md#enum-inferenceengine), [**Tryll::Client::ToolDef**](struct_tryll_1_1_client_1_1_tool_def.md), and the [**Tryll::Client::KnowledgePresentationConfig**](struct_tryll_1_1_client_1_1_knowledge_presentation_config.md) family.


All enum values mirror the FlatBuffers wire schema in `server/schema/messages.fbs`; edits here must be paired with the schema. 


    

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/tryll-mono/server/client-cpp/include/tryll/GraphDescription.h`

