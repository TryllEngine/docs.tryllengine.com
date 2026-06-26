

# Namespace Tryll::Client



[**Namespace List**](namespaces.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md)


















## Namespaces

| Type | Name |
| ---: | :--- |
| namespace | [**Internal**](namespace_tryll_1_1_client_1_1_internal.md) <br> |


## Classes

| Type | Name |
| ---: | :--- |
| class | [**AgentProxy**](class_tryll_1_1_client_1_1_agent_proxy.md) <br>_Client-side handle for one server-side agent._  |
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
| typedef std::function&lt; void(std::string\_view, std::uint64\_t, std::uint64\_t, float)&gt; | [**DownloadProgressCallback**](#typedef-downloadprogresscallback)  <br>_Callback invoked during_ [_**TryllClient::DownloadModel**_](class_tryll_1_1_client_1_1_tryll_client.md#function-downloadmodel) _as data arrives._ |
| typedef std::variant&lt; ::Tryll::NodeParams::GenerateParamsT, ::Tryll::NodeParams::HumanMessageGuardrailParamsT, ::Tryll::NodeParams::CannedResponseParamsT, ::Tryll::NodeParams::ToolCallParamsT, ::Tryll::NodeParams::RetrieveParamsT, ::Tryll::NodeParams::InstructionParamsT, ::Tryll::NodeParams::ClassifyIntentParamsT, ::Tryll::NodeParams::ClassifyIntentLLMParamsT, ::Tryll::NodeParams::IntentToInstructionParamsT, ::Tryll::NodeParams::GenerateAndSpeakParamsT, ::Tryll::NodeParams::SpeakParamsT &gt; | [**NodeParamsVariant**](#typedef-nodeparamsvariant)  <br>_Typed variant discriminating all concrete node-parameter PODs._  |
















































## Public Types Documentation




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
    ::Tryll::NodeParams::HumanMessageGuardrailParamsT,
    ::Tryll::NodeParams::CannedResponseParamsT,
    ::Tryll::NodeParams::ToolCallParamsT,
    ::Tryll::NodeParams::RetrieveParamsT,
    ::Tryll::NodeParams::InstructionParamsT,
    ::Tryll::NodeParams::ClassifyIntentParamsT,
    ::Tryll::NodeParams::ClassifyIntentLLMParamsT,
    ::Tryll::NodeParams::IntentToInstructionParamsT,
    ::Tryll::NodeParams::GenerateAndSpeakParamsT,
    ::Tryll::NodeParams::SpeakParamsT
>;
```



Generated AddXxx methods build a [**NodeParamsVariant**](namespace_tryll_1_1_client.md#typedef-nodeparamsvariant) and store it in NodeDesc.params. The ClientCodec serialises the variant into the FlatBuffers NodeParams union. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/AgentProxy.h`

