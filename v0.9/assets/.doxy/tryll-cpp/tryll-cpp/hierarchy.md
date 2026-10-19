
# Class Hierarchy

This inheritance list is sorted roughly, but not completely, alphabetically:


* **class** [**Tryll::Client::AgentProxy**](class_tryll_1_1_client_1_1_agent_proxy.md) _Client-side handle for one server-side agent._ 
* **class** [**Tryll::Client::AgentVariables**](class_tryll_1_1_client_1_1_agent_variables.md) _Client-side mirror + deferred-flush batcher for one agent's declared variables._ 
* **class** [**Tryll::Client::ConnectedSession**](class_tryll_1_1_client_1_1_connected_session.md) _RAII pair: owns a_ [_**ManagedServer**_](class_tryll_1_1_client_1_1_managed_server.md) _child process and a connected_[_**TryllClient**_](class_tryll_1_1_client_1_1_tryll_client.md) _._
* **class** [**Tryll::Client::GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) _Fluent builder for a workflow graph description sent to the server._ 
* **class** [**Tryll::Client::GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) 
* **class** [**Tryll::Client::ManagedServer**](class_tryll_1_1_client_1_1_managed_server.md) _RAII handle around a child_ `tryll_server` _process._
* **class** [**Tryll::Client::MutableKb**](class_tryll_1_1_client_1_1_mutable_kb.md) _Experimental session-scoped mutable knowledge-base handle._ 
* **class** [**Tryll::Client::TryllClient**](class_tryll_1_1_client_1_1_tryll_client.md) _TCP session to the Tryll server._ 
* **class** [**Tryll::Client::VoiceInput**](class_tryll_1_1_client_1_1_voice_input.md) 
* **struct** [**Tryll::Client::AgentCreateOptions**](struct_tryll_1_1_client_1_1_agent_create_options.md) 
* **struct** [**Tryll::Client::AgentKvCacheStatus**](struct_tryll_1_1_client_1_1_agent_kv_cache_status.md)     
    * **struct** [**Tryll::Client::AgentKvCacheEvictResult**](struct_tryll_1_1_client_1_1_agent_kv_cache_evict_result.md) 
    * **struct** [**Tryll::Client::AgentKvCachePrefillResult**](struct_tryll_1_1_client_1_1_agent_kv_cache_prefill_result.md) 
* **struct** [**Tryll::Client::AgentProxy::DialogInteraction**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_dialog_interaction.md) 
* **struct** [**Tryll::Client::AgentProxy::IntentClassifiedEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_intent_classified_event.md) _Owning payload for_ `event_type="intent_classified"` _(embedding classifier)._
* **struct** [**Tryll::Client::AgentProxy::IntentLlmClassifiedEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_intent_llm_classified_event.md) _Owning payload for_ `event_type="intent_llm_classified"` _._
* **struct** [**Tryll::Client::AgentProxy::IntentLlmLabelProb**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_intent_llm_label_prob.md) _One label / first-token probability pair from ClassifyIntentLLM._ 
* **struct** [**Tryll::Client::AgentProxy::ToolCallEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_tool_call_event.md) _Owning tool-call event for deferred / async handling._ 
* **struct** [**Tryll::Client::AgentProxy::ToolResult**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_tool_result.md) _One model-visible result correlated to a tool\_call event's call\_id._ 
* **struct** [**Tryll::Client::AgentProxy::VisemeEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_viseme_event.md) 
* **struct** [**Tryll::Client::AgentVariableDecl**](struct_tryll_1_1_client_1_1_agent_variable_decl.md) 
* **struct** [**Tryll::Client::AgentVariables::PendingUpdate**](struct_tryll_1_1_client_1_1_agent_variables_1_1_pending_update.md) 
* **struct** [**Tryll::Client::AudioFormat**](struct_tryll_1_1_client_1_1_audio_format.md) 
* **struct** [**Tryll::Client::DialogFileLoadResult**](struct_tryll_1_1_client_1_1_dialog_file_load_result.md) 
* **struct** [**Tryll::Client::DialogFileSaveResult**](struct_tryll_1_1_client_1_1_dialog_file_save_result.md) 
* **struct** [**Tryll::Client::DialogImportResult**](struct_tryll_1_1_client_1_1_dialog_import_result.md) 
* **struct** [**Tryll::Client::DialogIntention**](struct_tryll_1_1_client_1_1_dialog_intention.md) 
* **struct** [**Tryll::Client::DialogInteractionSnapshot**](struct_tryll_1_1_client_1_1_dialog_interaction_snapshot.md) 
* **struct** [**Tryll::Client::DialogKnowledge**](struct_tryll_1_1_client_1_1_dialog_knowledge.md) 
* **struct** [**Tryll::Client::DialogKnowledgeChunk**](struct_tryll_1_1_client_1_1_dialog_knowledge_chunk.md) 
* **struct** [**Tryll::Client::DialogProjectionWindow**](struct_tryll_1_1_client_1_1_dialog_projection_window.md) 
* **struct** [**Tryll::Client::DialogSlot**](struct_tryll_1_1_client_1_1_dialog_slot.md) 
* **struct** [**Tryll::Client::DialogSnapshot**](struct_tryll_1_1_client_1_1_dialog_snapshot.md) 
* **struct** [**Tryll::Client::DialogSummary**](struct_tryll_1_1_client_1_1_dialog_summary.md) 
* **struct** [**Tryll::Client::DialogToolCall**](struct_tryll_1_1_client_1_1_dialog_tool_call.md) 
* **struct** [**Tryll::Client::DialogToolResult**](struct_tryll_1_1_client_1_1_dialog_tool_result.md) 
* **struct** [**Tryll::Client::GameMemoryLoadResult**](struct_tryll_1_1_client_1_1_game_memory_load_result.md) 
* **struct** [**Tryll::Client::GameMemorySaveResult**](struct_tryll_1_1_client_1_1_game_memory_save_result.md) 
* **struct** [**Tryll::Client::GameMemorySnapshot**](struct_tryll_1_1_client_1_1_game_memory_snapshot.md) 
* **struct** [**Tryll::Client::GraphDescription::NodeDesc**](struct_tryll_1_1_client_1_1_graph_description_1_1_node_desc.md) _Internal serialisable description of one node._ 
* **struct** [**Tryll::Client::ManagedServerOptions**](struct_tryll_1_1_client_1_1_managed_server_options.md) _Configuration passed to_ [_**ManagedServer::Start**_](class_tryll_1_1_client_1_1_managed_server.md#function-start) _._
* **struct** [**Tryll::Client::MessageResult**](struct_tryll_1_1_client_1_1_message_result.md) _Placeholder for per-turn metadata returned from SendText._ 
* **struct** [**Tryll::Client::MutableKbHit**](struct_tryll_1_1_client_1_1_mutable_kb_hit.md) 
* **struct** [**Tryll::Client::MutableKbImportResult**](struct_tryll_1_1_client_1_1_mutable_kb_import_result.md) 
* **struct** [**Tryll::Client::MutableKbInfo**](struct_tryll_1_1_client_1_1_mutable_kb_info.md) 
* **struct** [**Tryll::Client::MutableKbListPage**](struct_tryll_1_1_client_1_1_mutable_kb_list_page.md) 
* **struct** [**Tryll::Client::MutableKbRecord**](struct_tryll_1_1_client_1_1_mutable_kb_record.md) 
* **struct** [**Tryll::Client::MutableKbSnapshot**](struct_tryll_1_1_client_1_1_mutable_kb_snapshot.md) 
* **struct** [**Tryll::Client::MutableKbStats**](struct_tryll_1_1_client_1_1_mutable_kb_stats.md) 
* **struct** [**Tryll::Client::MutableKbUpsert**](struct_tryll_1_1_client_1_1_mutable_kb_upsert.md) 
* **struct** [**Tryll::Client::MutableKbWriteResult**](struct_tryll_1_1_client_1_1_mutable_kb_write_result.md) 
* **struct** [**Tryll::Client::TranscriptUpdate**](struct_tryll_1_1_client_1_1_transcript_update.md) _Transcript update delivered via the_ [_**VoiceInput**_](class_tryll_1_1_client_1_1_voice_input.md) _callback._
* **struct** [**Tryll::Client::TryllClient::DebugCommandResult**](struct_tryll_1_1_client_1_1_tryll_client_1_1_debug_command_result.md) 
* **struct** [**Tryll::Client::TryllClient::EmbeddedStorageInfo**](struct_tryll_1_1_client_1_1_tryll_client_1_1_embedded_storage_info.md) _Result returned by_ `CreateEmbeddedStringStorage` _variants._
* **struct** [**Tryll::Client::TryllClient::EngineSelection**](struct_tryll_1_1_client_1_1_tryll_client_1_1_engine_selection.md) 
* **struct** [**Tryll::Client::TryllClient::LoadModelResult**](struct_tryll_1_1_client_1_1_tryll_client_1_1_load_model_result.md) _Explicitly load and pin a model into memory._ 
* **struct** [**Tryll::Client::TryllClient::ModelPlacementHint**](struct_tryll_1_1_client_1_1_tryll_client_1_1_model_placement_hint.md) 
* **struct** [**Tryll::Client::TryllClient::SessionConfig**](struct_tryll_1_1_client_1_1_tryll_client_1_1_session_config.md) 
* **struct** [**Tryll::Client::UtteranceOptions**](struct_tryll_1_1_client_1_1_utterance_options.md) _Per-utterance options passed to BeginUtterance._ 
* **struct** [**Tryll::Client::VoiceInputConfig**](struct_tryll_1_1_client_1_1_voice_input_config.md) _Parameters for creating a_ [_**VoiceInput**_](class_tryll_1_1_client_1_1_voice_input.md) _handle._
* **struct** [**Tryll::Generated::V\_93f2389f6dbf::Fingerprint**](struct_tryll_1_1_generated_1_1_v__93f2389f6dbf_1_1_fingerprint.md) 
* **struct** [**Tryll::Client::AgentVariables::PendingEntry**](struct_tryll_1_1_client_1_1_agent_variables_1_1_pending_entry.md) 
* **class** **std::runtime_error**    
    * **class** [**Tryll::Client::TryllError**](class_tryll_1_1_client_1_1_tryll_error.md) _Exception thrown by_ [_**TryllClient**_](class_tryll_1_1_client_1_1_tryll_client.md) _and_[_**AgentProxy**_](class_tryll_1_1_client_1_1_agent_proxy.md) _on server errors or disconnects._

