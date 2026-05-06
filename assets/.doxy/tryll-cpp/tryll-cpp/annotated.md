
# Class List


Here are the classes, structs, unions and interfaces with brief descriptions:

* **namespace** [**Tryll**](namespace_tryll.md)     
    * **class** [**AgentProxy**](class_tryll_1_1_agent_proxy.md) _Client-side handle for one server-side agent._     
    * **namespace** [**Client**](namespace_tryll_1_1_client.md)     
        * **class** [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) _Fluent builder for a workflow graph description sent to the server._     
            * **struct** [**NodeDesc**](struct_tryll_1_1_client_1_1_graph_description_1_1_node_desc.md) _Internal serialisable description of one node._     
            * **struct** [**RouteDesc**](struct_tryll_1_1_client_1_1_graph_description_1_1_route_desc.md) _Internal serialisable description of one wire between two nodes._     
        * **class** [**ManagedServer**](class_tryll_1_1_client_1_1_managed_server.md) _RAII handle around a child_ `tryll_server` _process._    
        * **struct** [**ManagedServerOptions**](struct_tryll_1_1_client_1_1_managed_server_options.md) _Configuration passed to_ [_**ManagedServer::Start**_](class_tryll_1_1_client_1_1_managed_server.md#function-start) _._    
        * **struct** [**ModelInfo**](struct_tryll_1_1_client_1_1_model_info.md) _Summary information for one model returned by_ [_**Tryll::TryllClient::ListModels**_](class_tryll_1_1_tryll_client.md#function-listmodels) _._    
        * **struct** [**ToolDef**](struct_tryll_1_1_client_1_1_tool_def.md) _Definition of a callable tool for a ToolCall node._     
        * **struct** [**ToolParamDef**](struct_tryll_1_1_client_1_1_tool_param_def.md) _Description of one parameter of a tool definition._     
    * **class** [**ConnectedSession**](class_tryll_1_1_connected_session.md) _RAII pair: owns a_ [_**Client::ManagedServer**_](class_tryll_1_1_client_1_1_managed_server.md) _child process and a connected_[_**TryllClient**_](class_tryll_1_1_tryll_client.md) _._    
    * **struct** [**MessageResult**](struct_tryll_1_1_message_result.md) _Placeholder for per-turn metadata returned from SendText._ 
    * **class** [**TryllClient**](class_tryll_1_1_tryll_client.md) _TCP session to the Tryll server._     
        * **struct** [**EmbeddedStorageInfo**](struct_tryll_1_1_tryll_client_1_1_embedded_storage_info.md) _Result returned by_ `CreateEmbeddedStringStorage` _variants._    
    * **class** [**TryllError**](class_tryll_1_1_tryll_error.md) _Exception thrown by_ [_**TryllClient**_](class_tryll_1_1_tryll_client.md) _and_[_**AgentProxy**_](class_tryll_1_1_agent_proxy.md) _on server errors or disconnects._    
* **namespace** [**std**](namespacestd.md) 

