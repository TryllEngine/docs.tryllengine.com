

# Namespace Tryll::Client



[**Namespace List**](namespaces.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md)




















## Classes

| Type | Name |
| ---: | :--- |
| class | [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) <br>_Fluent builder for a workflow graph description sent to the server._  |
| class | [**ManagedServer**](class_tryll_1_1_client_1_1_managed_server.md) <br>_RAII handle around a child_ `tryll_server` _process._ |
| struct | [**ManagedServerOptions**](struct_tryll_1_1_client_1_1_managed_server_options.md) <br>_Configuration passed to_ [_**ManagedServer::Start**_](class_tryll_1_1_client_1_1_managed_server.md#function-start) _._ |
| struct | [**ModelInfo**](struct_tryll_1_1_client_1_1_model_info.md) <br>_Summary information for one model returned by_ [_**Tryll::TryllClient::ListModels**_](class_tryll_1_1_tryll_client.md#function-listmodels) _._ |
| struct | [**ToolDef**](struct_tryll_1_1_client_1_1_tool_def.md) <br>_Definition of a callable tool for a ToolCall node._  |
| struct | [**ToolParamDef**](struct_tryll_1_1_client_1_1_tool_param_def.md) <br>_Description of one parameter of a tool definition._  |


## Public Types

| Type | Name |
| ---: | :--- |
| typedef std::function&lt; void(std::string\_view, std::uint64\_t, std::uint64\_t, float)&gt; | [**DownloadProgressCallback**](#typedef-downloadprogresscallback)  <br>_Callback invoked during_ [_**Tryll::TryllClient::DownloadModel**_](class_tryll_1_1_tryll_client.md#function-downloadmodel) _as data arrives._ |
| enum std::uint8\_t | [**InferenceEngine**](#enum-inferenceengine)  <br>_Inference backend identifiers._  |
| enum std::uint8\_t | [**ModelStatus**](#enum-modelstatus)  <br>_Status of a model on the server side._  |
| enum std::uint8\_t | [**NodeType**](#enum-nodetype)  <br>_Workflow node kinds recognised by the server._  |
















































## Public Types Documentation




### typedef DownloadProgressCallback 

_Callback invoked during_ [_**Tryll::TryllClient::DownloadModel**_](class_tryll_1_1_tryll_client.md#function-downloadmodel) _as data arrives._
```C++
using Tryll::Client::DownloadProgressCallback = 
std::function<void(std::string_view, std::uint64_t, std::uint64_t, float)>;
```



Invoked synchronously on the reader thread — keep the callback short and non-blocking.


Parameters (in order): `modelName`, `bytesDownloaded`, `totalBytes`, `percent` (0–100). 


        

<hr>



### enum InferenceEngine 

_Inference backend identifiers._ 
```C++
enum Tryll::Client::InferenceEngine {
    Mock = 0,
    LlamaCpp = 1,
    OnnxGenAI = 2,
    WindowsML = 3,
    OpenVino = 4,
    TensorRtLlm = 5
};
```



Must match the FlatBuffers `InferenceEngine` enum and `Tryll::Workflow::InferenceEngine`. Selected once per session via [**Tryll::TryllClient::ConfigureSession**](class_tryll_1_1_tryll_client.md#function-configuresession). 


        

<hr>



### enum ModelStatus 

_Status of a model on the server side._ 
```C++
enum Tryll::Client::ModelStatus {
    Absent = 0,
    Local = 1,
    Downloading = 2,
    Loaded = 3,
    Downloaded = 4
};
```



Mirrors the FlatBuffers `ModelStatus` enum. 


        

<hr>



### enum NodeType 

_Workflow node kinds recognised by the server._ 
```C++
enum Tryll::Client::NodeType {
    Generate = 0,
    HumanMessageGuardrail = 1,
    CannedResponse = 2,
    ToolCall = 3,
    Retrieve = 4,
    Instruction = 5
};
```



Must match the FlatBuffers `NodeType` enum. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo/server/client-cpp/include/tryll/GraphDescription.h`

