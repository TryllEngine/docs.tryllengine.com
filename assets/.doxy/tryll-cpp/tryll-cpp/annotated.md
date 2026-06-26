
# Class List


Here are the classes, structs, unions and interfaces with brief descriptions:

* **namespace** [**@347312275156240164130314360330001231161026003162**](namespace_0d347312275156240164130314360330001231161026003162.md) 
* **namespace** [**Tryll**](namespace_tryll.md)     
    * **namespace** [**Client**](namespace_tryll_1_1_client.md)     
        * **class** [**AgentProxy**](class_tryll_1_1_client_1_1_agent_proxy.md) _Client-side handle for one server-side agent._     
        * **struct** [**AudioFormat**](struct_tryll_1_1_client_1_1_audio_format.md)     
        * **class** [**ConnectedSession**](class_tryll_1_1_client_1_1_connected_session.md) _RAII pair: owns a_ [_**ManagedServer**_](class_tryll_1_1_client_1_1_managed_server.md) _child process and a connected_[_**TryllClient**_](class_tryll_1_1_client_1_1_tryll_client.md) _._    
        * **class** [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) _Fluent builder for a workflow graph description sent to the server._     
            * **struct** [**NodeDesc**](struct_tryll_1_1_client_1_1_graph_description_1_1_node_desc.md) _Internal serialisable description of one node._     
        * **class** [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md)     
        * **namespace** [**Internal**](namespace_tryll_1_1_client_1_1_internal.md) 
        * **class** [**ManagedServer**](class_tryll_1_1_client_1_1_managed_server.md) _RAII handle around a child_ `tryll_server` _process._    
        * **struct** [**ManagedServerOptions**](struct_tryll_1_1_client_1_1_managed_server_options.md) _Configuration passed to_ [_**ManagedServer::Start**_](class_tryll_1_1_client_1_1_managed_server.md#function-start) _._    
        * **struct** [**MessageResult**](struct_tryll_1_1_client_1_1_message_result.md) _Placeholder for per-turn metadata returned from SendText._ 
        * **struct** [**TranscriptUpdate**](struct_tryll_1_1_client_1_1_transcript_update.md) _Transcript update delivered via the_ [_**VoiceInput**_](class_tryll_1_1_client_1_1_voice_input.md) _callback._    
        * **class** [**TryllClient**](class_tryll_1_1_client_1_1_tryll_client.md) _TCP session to the Tryll server._     
            * **struct** [**DebugCommandResult**](struct_tryll_1_1_client_1_1_tryll_client_1_1_debug_command_result.md)     
            * **struct** [**EmbeddedStorageInfo**](struct_tryll_1_1_client_1_1_tryll_client_1_1_embedded_storage_info.md) _Result returned by_ `CreateEmbeddedStringStorage` _variants._    
            * **struct** [**SessionConfig**](struct_tryll_1_1_client_1_1_tryll_client_1_1_session_config.md) _Per-model-kind session configuration._     
        * **class** [**TryllError**](class_tryll_1_1_client_1_1_tryll_error.md) _Exception thrown by_ [_**TryllClient**_](class_tryll_1_1_client_1_1_tryll_client.md) _and_[_**AgentProxy**_](class_tryll_1_1_client_1_1_agent_proxy.md) _on server errors or disconnects._    
        * **struct** [**UtteranceOptions**](struct_tryll_1_1_client_1_1_utterance_options.md) _Per-utterance options passed to BeginUtterance._     
        * **class** [**VoiceInput**](class_tryll_1_1_client_1_1_voice_input.md)     
        * **struct** [**VoiceInputConfig**](struct_tryll_1_1_client_1_1_voice_input_config.md) _Parameters for creating a_ [_**VoiceInput**_](class_tryll_1_1_client_1_1_voice_input.md) _handle._    
    * **namespace** [**Generated**](namespace_tryll_1_1_generated.md)     
        * **namespace** [**V\_88c2c81422c1**](namespace_tryll_1_1_generated_1_1_v__88c2c81422c1.md)     
            * **struct** [**Fingerprint**](struct_tryll_1_1_generated_1_1_v__88c2c81422c1_1_1_fingerprint.md) 
* **namespace** [**std**](namespacestd.md) 

