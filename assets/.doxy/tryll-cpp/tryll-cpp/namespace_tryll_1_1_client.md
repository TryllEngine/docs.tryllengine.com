

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
| class | [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) <br>_Fluent builder for a workflow graph description sent to the server._  |
| class | [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) <br> |
| class | [**ManagedServer**](class_tryll_1_1_client_1_1_managed_server.md) <br>_RAII handle around a child_ `tryll_server` _process._ |
| struct | [**ManagedServerOptions**](struct_tryll_1_1_client_1_1_managed_server_options.md) <br>_Configuration passed to_ [_**ManagedServer::Start**_](class_tryll_1_1_client_1_1_managed_server.md#function-start) _._ |
| struct | [**MessageResult**](struct_tryll_1_1_client_1_1_message_result.md) <br>_Placeholder for per-turn metadata returned from SendText._  |
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
| typedef std::function&lt; void(std::string\_view, std::uint64\_t, std::uint64\_t, float)&gt; | [**DownloadProgressCallback**](#typedef-downloadprogresscallback)  <br>_Callback invoked during_ [_**TryllClient::DownloadModel**_](class_tryll_1_1_client_1_1_tryll_client.md#function-downloadmodel) _as data arrives._ |
| typedef std::variant&lt; ::Tryll::NodeParams::GenerateParamsT, ::Tryll::NodeParams::RegexGuardrailParamsT, ::Tryll::NodeParams::CannedResponseParamsT, ::Tryll::NodeParams::ToolCallParamsT, ::Tryll::NodeParams::RetrieveParamsT, ::Tryll::NodeParams::InstructionParamsT, ::Tryll::NodeParams::ClassifyIntentParamsT, ::Tryll::NodeParams::ClassifyIntentLLMParamsT, ::Tryll::NodeParams::IntentToInstructionParamsT, ::Tryll::NodeParams::GenerateAndSpeakParamsT, ::Tryll::NodeParams::SpeakParamsT, ::Tryll::NodeParams::PauseParamsT, ::Tryll::NodeParams::TransformParamsT &gt; | [**NodeParamsVariant**](#typedef-nodeparamsvariant)  <br>_Typed variant discriminating all concrete node-parameter PODs._  |
| enum std::uint8\_t | [**VariableType**](#enum-variabletype)  <br>_Type tag for a declared agent variable._  |
| typedef std::variant&lt; std::int64\_t, double, std::string, bool, std::vector&lt; std::string &gt; &gt; | [**VariableValue**](#typedef-variablevalue)  <br> |
















































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



### typedef DownloadProgressCallback 

_Callback invoked during_ [_**TryllClient::DownloadModel**_](class_tryll_1_1_client_1_1_tryll_client.md#function-downloadmodel) _as data arrives._
```C++
using Tryll::Client::DownloadProgressCallback = 
std::function<void(std::string_view, std::uint64_t, std::uint64_t, float)>;
```



Invoked synchronously on the reader thread — keep the callback short and non-blocking.


Parameters (in order): `modelName`, `bytesDownloaded`, `totalBytes`, `percent` (0–100). 


        

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
    ::Tryll::NodeParams::TransformParamsT
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

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/cpp/include/tryll/AgentCreateOptions.h`

