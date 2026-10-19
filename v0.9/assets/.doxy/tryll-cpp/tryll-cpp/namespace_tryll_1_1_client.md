

# Namespace Tryll::Client



[**Namespace List**](namespaces.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md)


















## Namespaces

| Type | Name |
| ---: | :--- |
| namespace | [**Internal**](namespace_tryll_1_1_client_1_1_internal.md) <br> |


## Classes

| Type | Name |
| ---: | :--- |
| struct | [**AgentCreateOptions**](struct_tryll_1_1_client_1_1_agent_create_options.md) <br> |
| struct | [**AgentKvCacheEvictResult**](struct_tryll_1_1_client_1_1_agent_kv_cache_evict_result.md) <br> |
| struct | [**AgentKvCachePrefillResult**](struct_tryll_1_1_client_1_1_agent_kv_cache_prefill_result.md) <br> |
| struct | [**AgentKvCacheStatus**](struct_tryll_1_1_client_1_1_agent_kv_cache_status.md) <br> |
| class | [**AgentProxy**](class_tryll_1_1_client_1_1_agent_proxy.md) <br>_Client-side handle for one server-side agent._  |
| struct | [**AgentVariableDecl**](struct_tryll_1_1_client_1_1_agent_variable_decl.md) <br> |
| class | [**AgentVariables**](class_tryll_1_1_client_1_1_agent_variables.md) <br>_Client-side mirror + deferred-flush batcher for one agent's declared variables._  |
| struct | [**AudioFormat**](struct_tryll_1_1_client_1_1_audio_format.md) <br> |
| class | [**ConnectedSession**](class_tryll_1_1_client_1_1_connected_session.md) <br>_RAII pair: owns a_ [_**ManagedServer**_](class_tryll_1_1_client_1_1_managed_server.md) _child process and a connected_[_**TryllClient**_](class_tryll_1_1_client_1_1_tryll_client.md) _._ |
| struct | [**DialogFileLoadResult**](struct_tryll_1_1_client_1_1_dialog_file_load_result.md) <br> |
| struct | [**DialogFileSaveResult**](struct_tryll_1_1_client_1_1_dialog_file_save_result.md) <br> |
| struct | [**DialogImportResult**](struct_tryll_1_1_client_1_1_dialog_import_result.md) <br> |
| struct | [**DialogIntention**](struct_tryll_1_1_client_1_1_dialog_intention.md) <br> |
| struct | [**DialogInteractionSnapshot**](struct_tryll_1_1_client_1_1_dialog_interaction_snapshot.md) <br> |
| struct | [**DialogKnowledge**](struct_tryll_1_1_client_1_1_dialog_knowledge.md) <br> |
| struct | [**DialogKnowledgeChunk**](struct_tryll_1_1_client_1_1_dialog_knowledge_chunk.md) <br> |
| struct | [**DialogProjectionWindow**](struct_tryll_1_1_client_1_1_dialog_projection_window.md) <br> |
| struct | [**DialogSlot**](struct_tryll_1_1_client_1_1_dialog_slot.md) <br> |
| struct | [**DialogSnapshot**](struct_tryll_1_1_client_1_1_dialog_snapshot.md) <br> |
| struct | [**DialogSummary**](struct_tryll_1_1_client_1_1_dialog_summary.md) <br> |
| struct | [**DialogToolCall**](struct_tryll_1_1_client_1_1_dialog_tool_call.md) <br> |
| struct | [**DialogToolResult**](struct_tryll_1_1_client_1_1_dialog_tool_result.md) <br> |
| struct | [**GameMemoryLoadResult**](struct_tryll_1_1_client_1_1_game_memory_load_result.md) <br> |
| struct | [**GameMemorySaveResult**](struct_tryll_1_1_client_1_1_game_memory_save_result.md) <br> |
| struct | [**GameMemorySnapshot**](struct_tryll_1_1_client_1_1_game_memory_snapshot.md) <br> |
| class | [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) <br>_Fluent builder for a workflow graph description sent to the server._  |
| class | [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) <br> |
| class | [**ManagedServer**](class_tryll_1_1_client_1_1_managed_server.md) <br>_RAII handle around a child_ `tryll_server` _process._ |
| struct | [**ManagedServerOptions**](struct_tryll_1_1_client_1_1_managed_server_options.md) <br>_Configuration passed to_ [_**ManagedServer::Start**_](class_tryll_1_1_client_1_1_managed_server.md#function-start) _._ |
| struct | [**MessageResult**](struct_tryll_1_1_client_1_1_message_result.md) <br>_Placeholder for per-turn metadata returned from SendText._  |
| class | [**MutableKb**](class_tryll_1_1_client_1_1_mutable_kb.md) <br>_Experimental session-scoped mutable knowledge-base handle._  |
| struct | [**MutableKbHit**](struct_tryll_1_1_client_1_1_mutable_kb_hit.md) <br> |
| struct | [**MutableKbImportResult**](struct_tryll_1_1_client_1_1_mutable_kb_import_result.md) <br> |
| struct | [**MutableKbInfo**](struct_tryll_1_1_client_1_1_mutable_kb_info.md) <br> |
| struct | [**MutableKbListPage**](struct_tryll_1_1_client_1_1_mutable_kb_list_page.md) <br> |
| struct | [**MutableKbRecord**](struct_tryll_1_1_client_1_1_mutable_kb_record.md) <br> |
| struct | [**MutableKbSnapshot**](struct_tryll_1_1_client_1_1_mutable_kb_snapshot.md) <br> |
| struct | [**MutableKbStats**](struct_tryll_1_1_client_1_1_mutable_kb_stats.md) <br> |
| struct | [**MutableKbUpsert**](struct_tryll_1_1_client_1_1_mutable_kb_upsert.md) <br> |
| struct | [**MutableKbWriteResult**](struct_tryll_1_1_client_1_1_mutable_kb_write_result.md) <br> |
| struct | [**TranscriptUpdate**](struct_tryll_1_1_client_1_1_transcript_update.md) <br>_Transcript update delivered via the_ [_**VoiceInput**_](class_tryll_1_1_client_1_1_voice_input.md) _callback._ |
| class | [**TryllClient**](class_tryll_1_1_client_1_1_tryll_client.md) <br>_TCP session to the Tryll server._  |
| class | [**TryllError**](class_tryll_1_1_client_1_1_tryll_error.md) <br>_Exception thrown by_ [_**TryllClient**_](class_tryll_1_1_client_1_1_tryll_client.md) _and_[_**AgentProxy**_](class_tryll_1_1_client_1_1_agent_proxy.md) _on server errors or disconnects._ |
| struct | [**UtteranceOptions**](struct_tryll_1_1_client_1_1_utterance_options.md) <br>_Per-utterance options passed to BeginUtterance._  |
| class | [**VoiceInput**](class_tryll_1_1_client_1_1_voice_input.md) <br> |
| struct | [**VoiceInputConfig**](struct_tryll_1_1_client_1_1_voice_input_config.md) <br>_Parameters for creating a_ [_**VoiceInput**_](class_tryll_1_1_client_1_1_voice_input.md) _handle._ |


## Public Types

| Type | Name |
| ---: | :--- |
| enum std::uint8\_t | [**AgentKvCachePrefixStatus**](#enum-agentkvcacheprefixstatus)  <br> |
| enum std::uint8\_t | [**AgentKvCacheResidency**](#enum-agentkvcacheresidency)  <br> |
| enum std::uint8\_t | [**DialogCompatibility**](#enum-dialogcompatibility)  <br> |
| typedef std::variant&lt; [**DialogSlot**](struct_tryll_1_1_client_1_1_dialog_slot.md), [**DialogKnowledge**](struct_tryll_1_1_client_1_1_dialog_knowledge.md), [**DialogIntention**](struct_tryll_1_1_client_1_1_dialog_intention.md), [**DialogToolCall**](struct_tryll_1_1_client_1_1_dialog_tool_call.md), [**DialogToolResult**](struct_tryll_1_1_client_1_1_dialog_tool_result.md), [**DialogSummary**](struct_tryll_1_1_client_1_1_dialog_summary.md) &gt; | [**DialogComponent**](#typedef-dialogcomponent)  <br> |
| enum std::uint8\_t | [**DialogSlotKind**](#enum-dialogslotkind)  <br> |
| typedef std::function&lt; void(std::string\_view, std::uint64\_t, std::uint64\_t, float)&gt; | [**DownloadProgressCallback**](#typedef-downloadprogresscallback)  <br>_Callback invoked during_ [_**TryllClient::DownloadModel**_](class_tryll_1_1_client_1_1_tryll_client.md#function-downloadmodel) _as data arrives._ |
| enum std::uint8\_t | [**MutableKbCompatibility**](#enum-mutablekbcompatibility)  <br> |
| enum std::uint8\_t | [**MutableKbEmbedTiming**](#enum-mutablekbembedtiming)  <br> |
| typedef std::variant&lt; ::Tryll::NodeParams::GenerateParamsT, ::Tryll::NodeParams::RegexGuardrailParamsT, ::Tryll::NodeParams::CannedResponseParamsT, ::Tryll::NodeParams::ToolCallParamsT, ::Tryll::NodeParams::RetrieveParamsT, ::Tryll::NodeParams::InstructionParamsT, ::Tryll::NodeParams::ClassifyIntentParamsT, ::Tryll::NodeParams::ClassifyIntentLLMParamsT, ::Tryll::NodeParams::IntentToInstructionParamsT, ::Tryll::NodeParams::GenerateAndSpeakParamsT, ::Tryll::NodeParams::SpeakParamsT, ::Tryll::NodeParams::PauseParamsT, ::Tryll::NodeParams::TransformParamsT, ::Tryll::NodeParams::BranchParamsT, ::Tryll::NodeParams::WriteToMemoryParamsT, ::Tryll::NodeParams::SummarizeParamsT, ::Tryll::NodeParams::ClassifyAttributesLLMParamsT &gt; | [**NodeParamsVariant**](#typedef-nodeparamsvariant)  <br>_Typed variant discriminating all concrete node-parameter PODs._  |
| enum std::uint8\_t | [**VariableType**](#enum-variabletype)  <br>_Type tag for a declared agent variable._  |
| typedef std::variant&lt; std::int64\_t, double, std::string, bool, std::vector&lt; std::string &gt; &gt; | [**VariableValue**](#typedef-variablevalue)  <br> |




## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::uint32\_t | [**kGameMemoryFormatVersion**](#variable-kgamememoryformatversion)   = `1`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  [**DialogSnapshot**](struct_tryll_1_1_client_1_1_dialog_snapshot.md) | [**DecodeFlatBuffer**](#function-decodeflatbuffer) (std::span&lt; const std::uint8\_t &gt; bytes) <br> |
|  [**GameMemorySnapshot**](struct_tryll_1_1_client_1_1_game_memory_snapshot.md) | [**DecodeGameMemory**](#function-decodegamememory) (std::span&lt; const std::uint8\_t &gt; bytes) <br> |
|  [**MutableKbSnapshot**](struct_tryll_1_1_client_1_1_mutable_kb_snapshot.md) | [**DecodeMutableKbSnapshot**](#function-decodemutablekbsnapshot) (std::span&lt; const std::uint8\_t &gt; bytes) <br> |
|  std::vector&lt; std::uint8\_t &gt; | [**EncodeFlatBuffer**](#function-encodeflatbuffer) (const [**DialogSnapshot**](struct_tryll_1_1_client_1_1_dialog_snapshot.md) & snapshot) <br>_Canonical binary form (file identifier "TDLG"). Throws_ [_**TryllError**_](class_tryll_1_1_client_1_1_tryll_error.md) _on failure._ |
|  std::vector&lt; std::uint8\_t &gt; | [**EncodeFlatBuffer**](#function-encodeflatbuffer) (const [**MutableKbSnapshot**](struct_tryll_1_1_client_1_1_mutable_kb_snapshot.md) & snapshot) <br>_Canonical TMKB bytes. Throws_ [_**TryllError**_](class_tryll_1_1_client_1_1_tryll_error.md) _on failure._ |
|  std::vector&lt; std::uint8\_t &gt; | [**EncodeFlatBuffer**](#function-encodeflatbuffer) (const [**GameMemorySnapshot**](struct_tryll_1_1_client_1_1_game_memory_snapshot.md) & snapshot) <br> |
|  flatbuffers::Offset&lt;::Tryll::GameMemory &gt; | [**PackGameMemory**](#function-packgamememory) (flatbuffers::FlatBufferBuilder & fbb, const [**GameMemorySnapshot**](struct_tryll_1_1_client_1_1_game_memory_snapshot.md) & snapshot) <br> |
|  flatbuffers::Offset&lt;::Tryll::PersistedDialogSnapshot &gt; | [**PackPersistedDialogSnapshot**](#function-packpersisteddialogsnapshot) (flatbuffers::FlatBufferBuilder & fbb, const [**DialogSnapshot**](struct_tryll_1_1_client_1_1_dialog_snapshot.md) & snapshot) <br> |
|  flatbuffers::Offset&lt;::Tryll::PersistedMutableKb &gt; | [**PackPersistedMutableKb**](#function-packpersistedmutablekb) (flatbuffers::FlatBufferBuilder & fbb, const [**MutableKbSnapshot**](struct_tryll_1_1_client_1_1_mutable_kb_snapshot.md) & snapshot) <br> |
|  [**GameMemorySnapshot**](struct_tryll_1_1_client_1_1_game_memory_snapshot.md) | [**UnpackGameMemory**](#function-unpackgamememory) (const ::Tryll::GameMemory & table) <br> |
|  [**DialogSnapshot**](struct_tryll_1_1_client_1_1_dialog_snapshot.md) | [**UnpackPersistedDialogSnapshot**](#function-unpackpersisteddialogsnapshot) (const ::Tryll::PersistedDialogSnapshot & table) <br> |
|  [**MutableKbSnapshot**](struct_tryll_1_1_client_1_1_mutable_kb_snapshot.md) | [**UnpackPersistedMutableKb**](#function-unpackpersistedmutablekb) (const ::Tryll::PersistedMutableKb & table) <br> |




























## Public Types Documentation




### enum AgentKvCachePrefixStatus 

```C++
enum Tryll::Client::AgentKvCachePrefixStatus {
    NotCurrent,
    Current
};
```




<hr>



### enum AgentKvCacheResidency 

```C++
enum Tryll::Client::AgentKvCacheResidency {
    Evicted,
    Resident
};
```




<hr>



### enum DialogCompatibility 

```C++
enum Tryll::Client::DialogCompatibility {
    Strict = 0,
    AllowGraphMismatch = 1
};
```




<hr>



### typedef DialogComponent 

```C++
using Tryll::Client::DialogComponent = std::variant<
   DialogSlot,
   DialogKnowledge,
   DialogIntention,
   DialogToolCall,
   DialogToolResult,
   DialogSummary>;
```




<hr>



### enum DialogSlotKind 

```C++
enum Tryll::Client::DialogSlotKind {
    User = 0,
    Instruction = 1,
    General = 2
};
```




<hr>



### typedef DownloadProgressCallback 

_Callback invoked during_ [_**TryllClient::DownloadModel**_](class_tryll_1_1_client_1_1_tryll_client.md#function-downloadmodel) _as data arrives._
```C++
using Tryll::Client::DownloadProgressCallback = 
std::function<void(std::string_view, std::uint64_t, std::uint64_t, float)>;
```



Invoked synchronously on the reader thread — keep the callback short and non-blocking.


Parameters (in order): `modelName`, `bytesDownloaded`, `totalBytes`, `percent` (0–100). 


        

<hr>



### enum MutableKbCompatibility 

```C++
enum Tryll::Client::MutableKbCompatibility {
    Strict = 0,
    AllowSchemaDrift = 1
};
```




<hr>



### enum MutableKbEmbedTiming 

```C++
enum Tryll::Client::MutableKbEmbedTiming {
    Unspecified = 0,
    Now = 1,
    Lazy = 2
};
```




<hr>



### typedef NodeParamsVariant 

_Typed variant discriminating all concrete node-parameter PODs._ 
```C++
using Tryll::Client::NodeParamsVariant =  std::variant<
    ::Tryll::NodeParams::GenerateParamsT,
    ::Tryll::NodeParams::RegexGuardrailParamsT,
    ::Tryll::NodeParams::CannedResponseParamsT,
    ::Tryll::NodeParams::ToolCallParamsT,
    ::Tryll::NodeParams::RetrieveParamsT,
    ::Tryll::NodeParams::InstructionParamsT,
    ::Tryll::NodeParams::ClassifyIntentParamsT,
    ::Tryll::NodeParams::ClassifyIntentLLMParamsT,
    ::Tryll::NodeParams::IntentToInstructionParamsT,
    ::Tryll::NodeParams::GenerateAndSpeakParamsT,
    ::Tryll::NodeParams::SpeakParamsT,
    ::Tryll::NodeParams::PauseParamsT,
    ::Tryll::NodeParams::TransformParamsT,
    ::Tryll::NodeParams::BranchParamsT,
    ::Tryll::NodeParams::WriteToMemoryParamsT,
    ::Tryll::NodeParams::SummarizeParamsT,
    ::Tryll::NodeParams::ClassifyAttributesLLMParamsT
>;
```



Generated AddXxx methods build a [**NodeParamsVariant**](namespace_tryll_1_1_client.md#typedef-nodeparamsvariant) and store it in NodeDesc.params. The ClientCodec serialises the variant into the FlatBuffers NodeParams union. 


        

<hr>



### enum VariableType 

_Type tag for a declared agent variable._ 
```C++
enum Tryll::Client::VariableType {
    Int,
    Float,
    String,
    Bool,
    StringSet
};
```




<hr>



### typedef VariableValue 

```C++
using Tryll::Client::VariableValue = std::variant<std::int64_t, double, std::string, bool, std::vector<std::string>>;
```



A variable's value. The active alternative fixes its type for the agent's whole lifetime (server-enforced; see VariableTypeMismatch 3014). 


        

<hr>
## Public Attributes Documentation




### variable kGameMemoryFormatVersion 

```C++
std::uint32_t Tryll::Client::kGameMemoryFormatVersion;
```




<hr>
## Public Functions Documentation




### function DecodeFlatBuffer 

```C++
DialogSnapshot Tryll::Client::DecodeFlatBuffer (
    std::span< const std::uint8_t > bytes
) 
```




<hr>



### function DecodeGameMemory 

```C++
GameMemorySnapshot Tryll::Client::DecodeGameMemory (
    std::span< const std::uint8_t > bytes
) 
```




<hr>



### function DecodeMutableKbSnapshot 

```C++
MutableKbSnapshot Tryll::Client::DecodeMutableKbSnapshot (
    std::span< const std::uint8_t > bytes
) 
```




<hr>



### function EncodeFlatBuffer 

_Canonical binary form (file identifier "TDLG"). Throws_ [_**TryllError**_](class_tryll_1_1_client_1_1_tryll_error.md) _on failure._
```C++
std::vector< std::uint8_t > Tryll::Client::EncodeFlatBuffer (
    const DialogSnapshot & snapshot
) 
```




<hr>



### function EncodeFlatBuffer 

_Canonical TMKB bytes. Throws_ [_**TryllError**_](class_tryll_1_1_client_1_1_tryll_error.md) _on failure._
```C++
std::vector< std::uint8_t > Tryll::Client::EncodeFlatBuffer (
    const MutableKbSnapshot & snapshot
) 
```




<hr>



### function EncodeFlatBuffer 

```C++
std::vector< std::uint8_t > Tryll::Client::EncodeFlatBuffer (
    const GameMemorySnapshot & snapshot
) 
```




<hr>



### function PackGameMemory 

```C++
flatbuffers::Offset<::Tryll::GameMemory > Tryll::Client::PackGameMemory (
    flatbuffers::FlatBufferBuilder & fbb,
    const GameMemorySnapshot & snapshot
) 
```




<hr>



### function PackPersistedDialogSnapshot 

```C++
flatbuffers::Offset<::Tryll::PersistedDialogSnapshot > Tryll::Client::PackPersistedDialogSnapshot (
    flatbuffers::FlatBufferBuilder & fbb,
    const DialogSnapshot & snapshot
) 
```




<hr>



### function PackPersistedMutableKb 

```C++
flatbuffers::Offset<::Tryll::PersistedMutableKb > Tryll::Client::PackPersistedMutableKb (
    flatbuffers::FlatBufferBuilder & fbb,
    const MutableKbSnapshot & snapshot
) 
```




<hr>



### function UnpackGameMemory 

```C++
GameMemorySnapshot Tryll::Client::UnpackGameMemory (
    const ::Tryll::GameMemory & table
) 
```




<hr>



### function UnpackPersistedDialogSnapshot 

```C++
DialogSnapshot Tryll::Client::UnpackPersistedDialogSnapshot (
    const ::Tryll::PersistedDialogSnapshot & table
) 
```




<hr>



### function UnpackPersistedMutableKb 

```C++
MutableKbSnapshot Tryll::Client::UnpackPersistedMutableKb (
    const ::Tryll::PersistedMutableKb & table
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/cpp/include/tryll/AgentCreateOptions.h`

