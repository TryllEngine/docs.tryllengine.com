
# Class List


Here are the classes, structs, unions and interfaces with brief descriptions:

* **namespace** [**@221225266365143145043236145257124227201005334110**](namespace_0d221225266365143145043236145257124227201005334110.md) 
* **namespace** [**Tryll**](namespace_tryll.md)     
    * **namespace** [**Client**](namespace_tryll_1_1_client.md)     
        * **struct** [**AgentCreateOptions**](struct_tryll_1_1_client_1_1_agent_create_options.md)     
        * **struct** [**AgentKvCacheEvictResult**](struct_tryll_1_1_client_1_1_agent_kv_cache_evict_result.md)     
        * **struct** [**AgentKvCachePrefillResult**](struct_tryll_1_1_client_1_1_agent_kv_cache_prefill_result.md)     
        * **struct** [**AgentKvCacheStatus**](struct_tryll_1_1_client_1_1_agent_kv_cache_status.md)     
        * **class** [**AgentProxy**](class_tryll_1_1_client_1_1_agent_proxy.md) _Client-side handle for one server-side agent._     
            * **struct** [**DialogInteraction**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_dialog_interaction.md)     
            * **struct** [**IntentClassifiedEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_intent_classified_event.md) _Owning payload for_ `event_type="intent_classified"` _(embedding classifier)._    
            * **struct** [**IntentLlmClassifiedEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_intent_llm_classified_event.md) _Owning payload for_ `event_type="intent_llm_classified"` _._    
            * **struct** [**IntentLlmLabelProb**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_intent_llm_label_prob.md) _One label / first-token probability pair from ClassifyIntentLLM._     
            * **struct** [**ToolCallEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_tool_call_event.md) _Owning tool-call event for deferred / async handling._     
            * **struct** [**ToolResult**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_tool_result.md) _One model-visible result correlated to a tool\_call event's call\_id._     
            * **struct** [**VisemeEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_viseme_event.md)     
        * **struct** [**AgentVariableDecl**](struct_tryll_1_1_client_1_1_agent_variable_decl.md)     
        * **class** [**AgentVariables**](class_tryll_1_1_client_1_1_agent_variables.md) _Client-side mirror + deferred-flush batcher for one agent's declared variables._     
            * **struct** [**PendingUpdate**](struct_tryll_1_1_client_1_1_agent_variables_1_1_pending_update.md)     
        * **struct** [**AudioFormat**](struct_tryll_1_1_client_1_1_audio_format.md)     
        * **class** [**ConnectedSession**](class_tryll_1_1_client_1_1_connected_session.md) _RAII pair: owns a_ [_**ManagedServer**_](class_tryll_1_1_client_1_1_managed_server.md) _child process and a connected_[_**TryllClient**_](class_tryll_1_1_client_1_1_tryll_client.md) _._    
        * **struct** [**DialogFileLoadResult**](struct_tryll_1_1_client_1_1_dialog_file_load_result.md)     
        * **struct** [**DialogFileSaveResult**](struct_tryll_1_1_client_1_1_dialog_file_save_result.md)     
        * **struct** [**DialogImportResult**](struct_tryll_1_1_client_1_1_dialog_import_result.md)     
        * **struct** [**DialogIntention**](struct_tryll_1_1_client_1_1_dialog_intention.md)     
        * **struct** [**DialogInteractionSnapshot**](struct_tryll_1_1_client_1_1_dialog_interaction_snapshot.md)     
        * **struct** [**DialogKnowledge**](struct_tryll_1_1_client_1_1_dialog_knowledge.md)     
        * **struct** [**DialogKnowledgeChunk**](struct_tryll_1_1_client_1_1_dialog_knowledge_chunk.md)     
        * **struct** [**DialogProjectionWindow**](struct_tryll_1_1_client_1_1_dialog_projection_window.md)     
        * **struct** [**DialogSlot**](struct_tryll_1_1_client_1_1_dialog_slot.md)     
        * **struct** [**DialogSnapshot**](struct_tryll_1_1_client_1_1_dialog_snapshot.md)     
        * **struct** [**DialogSummary**](struct_tryll_1_1_client_1_1_dialog_summary.md)     
        * **struct** [**DialogToolCall**](struct_tryll_1_1_client_1_1_dialog_tool_call.md)     
        * **struct** [**DialogToolResult**](struct_tryll_1_1_client_1_1_dialog_tool_result.md)     
        * **struct** [**GameMemoryLoadResult**](struct_tryll_1_1_client_1_1_game_memory_load_result.md)     
        * **struct** [**GameMemorySaveResult**](struct_tryll_1_1_client_1_1_game_memory_save_result.md)     
        * **struct** [**GameMemorySnapshot**](struct_tryll_1_1_client_1_1_game_memory_snapshot.md)     
        * **class** [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) _Fluent builder for a workflow graph description sent to the server._     
            * **struct** [**NodeDesc**](struct_tryll_1_1_client_1_1_graph_description_1_1_node_desc.md) _Internal serialisable description of one node._     
        * **class** [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md)     
        * **namespace** [**Internal**](namespace_tryll_1_1_client_1_1_internal.md) 
        * **class** [**ManagedServer**](class_tryll_1_1_client_1_1_managed_server.md) _RAII handle around a child_ `tryll_server` _process._    
        * **struct** [**ManagedServerOptions**](struct_tryll_1_1_client_1_1_managed_server_options.md) _Configuration passed to_ [_**ManagedServer::Start**_](class_tryll_1_1_client_1_1_managed_server.md#function-start) _._    
        * **struct** [**MessageResult**](struct_tryll_1_1_client_1_1_message_result.md) _Placeholder for per-turn metadata returned from SendText._ 
        * **class** [**MutableKb**](class_tryll_1_1_client_1_1_mutable_kb.md) _Experimental session-scoped mutable knowledge-base handle._     
        * **struct** [**MutableKbHit**](struct_tryll_1_1_client_1_1_mutable_kb_hit.md)     
        * **struct** [**MutableKbImportResult**](struct_tryll_1_1_client_1_1_mutable_kb_import_result.md)     
        * **struct** [**MutableKbInfo**](struct_tryll_1_1_client_1_1_mutable_kb_info.md)     
        * **struct** [**MutableKbListPage**](struct_tryll_1_1_client_1_1_mutable_kb_list_page.md)     
        * **struct** [**MutableKbRecord**](struct_tryll_1_1_client_1_1_mutable_kb_record.md)     
        * **struct** [**MutableKbSnapshot**](struct_tryll_1_1_client_1_1_mutable_kb_snapshot.md)     
        * **struct** [**MutableKbStats**](struct_tryll_1_1_client_1_1_mutable_kb_stats.md)     
        * **struct** [**MutableKbUpsert**](struct_tryll_1_1_client_1_1_mutable_kb_upsert.md)     
        * **struct** [**MutableKbWriteResult**](struct_tryll_1_1_client_1_1_mutable_kb_write_result.md)     
        * **struct** [**TranscriptUpdate**](struct_tryll_1_1_client_1_1_transcript_update.md) _Transcript update delivered via the_ [_**VoiceInput**_](class_tryll_1_1_client_1_1_voice_input.md) _callback._    
        * **class** [**TryllClient**](class_tryll_1_1_client_1_1_tryll_client.md) _TCP session to the Tryll server._     
            * **struct** [**DebugCommandResult**](struct_tryll_1_1_client_1_1_tryll_client_1_1_debug_command_result.md)     
            * **struct** [**EmbeddedStorageInfo**](struct_tryll_1_1_client_1_1_tryll_client_1_1_embedded_storage_info.md) _Result returned by_ `CreateEmbeddedStringStorage` _variants._    
            * **struct** [**EngineSelection**](struct_tryll_1_1_client_1_1_tryll_client_1_1_engine_selection.md)     
            * **struct** [**LoadModelResult**](struct_tryll_1_1_client_1_1_tryll_client_1_1_load_model_result.md) _Explicitly load and pin a model into memory._     
            * **struct** [**ModelPlacementHint**](struct_tryll_1_1_client_1_1_tryll_client_1_1_model_placement_hint.md)     
            * **struct** [**SessionConfig**](struct_tryll_1_1_client_1_1_tryll_client_1_1_session_config.md)     
        * **class** [**TryllError**](class_tryll_1_1_client_1_1_tryll_error.md) _Exception thrown by_ [_**TryllClient**_](class_tryll_1_1_client_1_1_tryll_client.md) _and_[_**AgentProxy**_](class_tryll_1_1_client_1_1_agent_proxy.md) _on server errors or disconnects._    
        * **struct** [**UtteranceOptions**](struct_tryll_1_1_client_1_1_utterance_options.md) _Per-utterance options passed to BeginUtterance._     
        * **class** [**VoiceInput**](class_tryll_1_1_client_1_1_voice_input.md)     
        * **struct** [**VoiceInputConfig**](struct_tryll_1_1_client_1_1_voice_input_config.md) _Parameters for creating a_ [_**VoiceInput**_](class_tryll_1_1_client_1_1_voice_input.md) _handle._    
    * **namespace** [**Generated**](namespace_tryll_1_1_generated.md)     
        * **namespace** [**V\_93f2389f6dbf**](namespace_tryll_1_1_generated_1_1_v__93f2389f6dbf.md)     
            * **struct** [**Fingerprint**](struct_tryll_1_1_generated_1_1_v__93f2389f6dbf_1_1_fingerprint.md) 
* **struct** [**PendingEntry**](struct_tryll_1_1_client_1_1_agent_variables_1_1_pending_entry.md)     
* **namespace** [**std**](namespacestd.md) 

