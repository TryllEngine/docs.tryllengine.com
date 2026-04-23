
# Class Hierarchy

This inheritance list is sorted roughly, but not completely, alphabetically:


* **class** [**Tryll::AgentProxy**](class_tryll_1_1_agent_proxy.md) _Client-side handle for one server-side agent._ 
* **class** [**Tryll::Client::GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) _Fluent builder for a workflow graph description sent to the server._ 
* **class** [**Tryll::TryllClient**](class_tryll_1_1_tryll_client.md) _TCP session to the Tryll server._ 
* **struct** [**Tryll::Client::GraphDescription::NodeDesc**](struct_tryll_1_1_client_1_1_graph_description_1_1_node_desc.md) _Internal serialisable description of one node._ 
* **struct** [**Tryll::Client::GraphDescription::RouteDesc**](struct_tryll_1_1_client_1_1_graph_description_1_1_route_desc.md) _Internal serialisable description of one wire between two nodes._ 
* **struct** [**Tryll::Client::KnowledgePresentationConfig**](struct_tryll_1_1_client_1_1_knowledge_presentation_config.md) _Per-agent knowledge presentation configuration._ 
* **struct** [**Tryll::Client::ModelInfo**](struct_tryll_1_1_client_1_1_model_info.md) _Summary information for one model returned by_ [_**Tryll::TryllClient::ListModels**_](class_tryll_1_1_tryll_client.md#function-listmodels) _._
* **struct** [**Tryll::Client::RetrievePresentationConfig**](struct_tryll_1_1_client_1_1_retrieve_presentation_config.md) _Formatting rules for one knowledge source (or the default for all sources)._ 
* **struct** [**Tryll::Client::ToolDef**](struct_tryll_1_1_client_1_1_tool_def.md) _Definition of a callable tool for a ToolCall node._ 
* **struct** [**Tryll::Client::ToolParamDef**](struct_tryll_1_1_client_1_1_tool_param_def.md) _Description of one parameter of a tool definition._ 
* **struct** [**Tryll::MessageResult**](struct_tryll_1_1_message_result.md) _Placeholder for per-turn metadata returned from SendText._ 
* **struct** [**Tryll::TryllClient::EmbeddedStorageInfo**](struct_tryll_1_1_tryll_client_1_1_embedded_storage_info.md) _Result returned by_ `CreateEmbeddedStringStorage` _variants._
* **class** **std::runtime_error**    
    * **class** [**Tryll::TryllError**](class_tryll_1_1_tryll_error.md) _Exception thrown by_ [_**TryllClient**_](class_tryll_1_1_tryll_client.md) _and_[_**AgentProxy**_](class_tryll_1_1_agent_proxy.md) _on server errors or disconnects._

