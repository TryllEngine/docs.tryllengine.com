

# Class Tryll::TryllClient



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**TryllClient**](class_tryll_1_1_tryll_client.md)



_TCP session to the Tryll server._ [More...](#detailed-description)

* `#include <TryllClient.h>`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**EmbeddedStorageInfo**](struct_tryll_1_1_tryll_client_1_1_embedded_storage_info.md) <br>_Result returned by_ `CreateEmbeddedStringStorage` _variants._ |






















## Public Functions

| Type | Name |
| ---: | :--- |
|  void | [**ConfigureSession**](#function-configuresession) ([**Client::InferenceEngine**](namespace_tryll_1_1_client.md#enum-inferenceengine) engine, bool allowAutoModelDownloading=false, std::string\_view gameName={}, std::chrono::milliseconds timeout=std::chrono::seconds{30}) <br>_Select the session's inference engine._  |
|  [**AgentProxy**](class_tryll_1_1_agent_proxy.md) | [**CreateAgent**](#function-createagent) (const [**Tryll::Client::GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) & graph, bool enableDiagnostics=false, std::optional&lt; std::chrono::milliseconds &gt; timeout=std::nullopt) <br>_Blocking convenience wrapper over_ [_**CreateAgentAsync**_](class_tryll_1_1_tryll_client.md#function-createagentasync) _._ |
|  std::future&lt; [**AgentProxy**](class_tryll_1_1_agent_proxy.md) &gt; | [**CreateAgentAsync**](#function-createagentasync) (const [**Tryll::Client::GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) & graph, bool enableDiagnostics=false) <br>_Asynchronously create an agent with the given graph description._  |
|  [**EmbeddedStorageInfo**](struct_tryll_1_1_tryll_client_1_1_embedded_storage_info.md) | [**CreateEmbeddedStringStorage**](#function-createembeddedstringstorage) (std::string\_view name, std::string\_view configPath, std::string\_view embeddingModel={}, std::optional&lt; std::chrono::milliseconds &gt; timeout=std::nullopt) <br>_Create an_ `EmbeddedStringStorage` _from a server-side config._ |
|  [**EmbeddedStorageInfo**](struct_tryll_1_1_tryll_client_1_1_embedded_storage_info.md) | [**CreateEmbeddedStringStorageFromStrings**](#function-createembeddedstringstoragefromstrings) (std::string\_view name, const std::vector&lt; std::string &gt; & strings, std::string\_view embeddingModel, std::optional&lt; std::chrono::milliseconds &gt; timeout=std::nullopt) <br>_Create an_ `EmbeddedStringStorage` _from inline strings._ |
|  void | [**CreateStringStorage**](#function-createstringstorage) (std::string\_view name, const std::vector&lt; std::string &gt; & strings, std::chrono::milliseconds timeout=std::chrono::seconds{10}) <br>_Create a named_ `StringStorage` _on the server from an inline list._ |
|  void | [**CreateStringStorageFromFile**](#function-createstringstoragefromfile) (std::string\_view name, std::string\_view filePath, std::chrono::milliseconds timeout=std::chrono::seconds{10}) <br>_Create a named_ `StringStorage` _on the server from a server-side file._ |
|  void | [**DestroyEmbeddedStringStorage**](#function-destroyembeddedstringstorage) (std::string\_view name, std::chrono::milliseconds timeout=std::chrono::seconds{10}) <br>_Destroy a named_ `EmbeddedStringStorage` _._ |
|  void | [**DestroyStringStorage**](#function-destroystringstorage) (std::string\_view name, std::chrono::milliseconds timeout=std::chrono::seconds{10}) <br>_Destroy a named_ `StringStorage` _on the server._ |
|  void | [**DownloadModel**](#function-downloadmodel) (std::string\_view modelName, [**Client::DownloadProgressCallback**](namespace_tryll_1_1_client.md#typedef-downloadprogresscallback) onProgress=nullptr, std::chrono::milliseconds timeout=std::chrono::minutes{30}) <br>_Start downloading a model on the server and block until complete._  |
|  std::uint64\_t | [**GetSessionId**](#function-getsessionid) () noexcept const<br>_Server-assigned session identifier._  |
|  std::vector&lt; [**Client::ModelInfo**](struct_tryll_1_1_client_1_1_model_info.md) &gt; | [**ListModels**](#function-listmodels) (std::chrono::milliseconds timeout=std::chrono::seconds{10}) <br>_Request the list of models known to the server for the session's engine._  |
|  void | [**LoadModel**](#function-loadmodel) (std::string\_view modelName, std::chrono::milliseconds timeout=std::chrono::minutes{5}) <br>_Explicitly load and pin a model into memory._  |
|  void | [**Shutdown**](#function-shutdown) (std::chrono::milliseconds timeout=std::chrono::seconds{30}) <br>_Blocking convenience wrapper over_ [_**ShutdownAsync**_](class_tryll_1_1_tryll_client.md#function-shutdownasync) _._ |
|  std::future&lt; void &gt; | [**ShutdownAsync**](#function-shutdownasync) () <br>_Asynchronously shut down the connection._  |
|   | [**TryllClient**](#function-tryllclient-13) (const TryllClient &) = default<br> |
|   | [**TryllClient**](#function-tryllclient-23) (TryllClient &&) noexcept<br> |
|  void | [**UnloadModel**](#function-unloadmodel) (std::string\_view modelName, std::chrono::milliseconds timeout=std::chrono::seconds{30}) <br>_Unpin a previously pinned model._  |
|  TryllClient & | [**operator=**](#function-operator) (const TryllClient &) = default<br> |
|  TryllClient & | [**operator=**](#function-operator_1) (TryllClient &&) noexcept<br> |
|   | [**~TryllClient**](#function-tryllclient) () <br> |


## Public Static Functions

| Type | Name |
| ---: | :--- |
|  TryllClient | [**Connect**](#function-connect) (std::string\_view host, std::uint16\_t port, std::chrono::milliseconds timeout=std::chrono::seconds{30}) <br>_Blocking convenience wrapper over_ [_**ConnectAsync**_](class_tryll_1_1_tryll_client.md#function-connectasync) _._ |
|  std::future&lt; TryllClient &gt; | [**ConnectAsync**](#function-connectasync) (std::string\_view host, std::uint16\_t port) <br>_Asynchronously connect and wait for_ `SessionReady` _._ |
|  [**ConnectedSession**](class_tryll_1_1_connected_session.md) | [**RunAndConnect**](#function-runandconnect) ([**Client::ManagedServerOptions**](struct_tryll_1_1_client_1_1_managed_server_options.md) serverOpts, std::chrono::milliseconds connectTimeout=std::chrono::seconds{30}) <br>_Spawn a local server and connect to it — recommended for single-process deployments where the client owns the server lifecycle._  |
|  std::future&lt; [**ConnectedSession**](class_tryll_1_1_connected_session.md) &gt; | [**RunAndConnectAsync**](#function-runandconnectasync) ([**Client::ManagedServerOptions**](struct_tryll_1_1_client_1_1_managed_server_options.md) serverOpts) <br>_Asynchronous variant of_ [_**RunAndConnect**_](class_tryll_1_1_tryll_client.md#function-runandconnect) _._ |


























## Detailed Description


Copyable — all copies share the same underlying connection via `shared_ptr`. The connection stays open until the last copy goes out of scope or [**Shutdown**](class_tryll_1_1_tryll_client.md#function-shutdown) is called. 


    
## Public Functions Documentation




### function ConfigureSession 

_Select the session's inference engine._ 
```C++
void Tryll::TryllClient::ConfigureSession (
    Client::InferenceEngine engine,
    bool allowAutoModelDownloading=false,
    std::string_view gameName={},
    std::chrono::milliseconds timeout=std::chrono::seconds{30}
) 
```



Must be called after [**Connect**](class_tryll_1_1_tryll_client.md#function-connect) and before [**CreateAgent**](class_tryll_1_1_tryll_client.md#function-createagent) or any model-management call.




**Parameters:**


* `engine` Inference backend; see [**Client::InferenceEngine**](namespace_tryll_1_1_client.md#enum-inferenceengine). 
* `allowAutoModelDownloading` When `true`, the server automatically downloads any missing models referenced by a graph during [**CreateAgent**](class_tryll_1_1_tryll_client.md#function-createagent) instead of failing immediately. Intended for development and prototyping — not for production use. When this flag is set, [**CreateAgent**](class_tryll_1_1_tryll_client.md#function-createagent) and [**CreateAgentAsync**](class_tryll_1_1_tryll_client.md#function-createagentasync) use a 30-minute default timeout unless the caller overrides it explicitly. 
* `timeout` Maximum time to wait for the response.



**Exception:**


* `TryllError` On server-reported errors or timeout. 



**Parameters:**


* `gameName` Integration identifier for telemetry grouping (e.g. "my-game"). Nullable — omit for anonymous sessions. 




        

<hr>



### function CreateAgent 

_Blocking convenience wrapper over_ [_**CreateAgentAsync**_](class_tryll_1_1_tryll_client.md#function-createagentasync) _._
```C++
AgentProxy Tryll::TryllClient::CreateAgent (
    const Tryll::Client::GraphDescription & graph,
    bool enableDiagnostics=false,
    std::optional< std::chrono::milliseconds > timeout=std::nullopt
) 
```





**Parameters:**


* `graph` Fully-built graph description. 
* `enableDiagnostics` When `true`, the server serialises per-node execution data into `TurnComplete.debug_info` for every turn. Off by default; enable for QA/eval pipelines. 
* `timeout` Maximum time to wait for `CreateAgentResponse`. Defaults to 30 seconds normally, or to 30 minutes when [**ConfigureSession**](class_tryll_1_1_tryll_client.md#function-configuresession) was called with `allowAutoModelDownloading=true` (to accommodate model downloads).



**Returns:**

[**AgentProxy**](class_tryll_1_1_agent_proxy.md) bound to the new server-side agent.




**Exception:**


* `TryllError` On validation failures, server-reported errors (including `ModelAutoDownloadFailed` when `allowAutoModelDownloading=true` and a model cannot be downloaded), or timeout. 




        

<hr>



### function CreateAgentAsync 

_Asynchronously create an agent with the given graph description._ 
```C++
std::future< AgentProxy > Tryll::TryllClient::CreateAgentAsync (
    const Tryll::Client::GraphDescription & graph,
    bool enableDiagnostics=false
) 
```





**Parameters:**


* `graph` Fully-built graph description. 
* `enableDiagnostics` When `true`, the server serialises per-node execution data into `TurnComplete.debug_info` for every turn. Off by default for zero overhead in production; enable for QA/eval pipelines.



**Note:**

When [**ConfigureSession**](class_tryll_1_1_tryll_client.md#function-configuresession) was called with `allowAutoModelDownloading=true`, this call may block internally while models are being downloaded. The returned future completes when creation succeeds or fails definitively. `DownloadProgress` frames are emitted during the download (observable via the `DownloadModel` progress callback path).




**Returns:**

Future yielding an [**AgentProxy**](class_tryll_1_1_agent_proxy.md) bound to the new agent. Calling `future::get()` propagates any [**TryllError**](class_tryll_1_1_tryll_error.md) raised during validation or creation. 





        

<hr>



### function CreateEmbeddedStringStorage 

_Create an_ `EmbeddedStringStorage` _from a server-side config._
```C++
EmbeddedStorageInfo Tryll::TryllClient::CreateEmbeddedStringStorage (
    std::string_view name,
    std::string_view configPath,
    std::string_view embeddingModel={},
    std::optional< std::chrono::milliseconds > timeout=std::nullopt
) 
```



Path A: the server loads records and (optionally) an on-disk HNSW index described by a `*`.json config referencing a `*`.kb.json records file.


If `allow_auto_model_downloading` was set in `ConfigureSession` and the required embedding model is absent, it will be downloaded before the storage is built. In that case the default timeout is automatically raised to 30 minutes; pass an explicit value to override.




**Parameters:**


* `name` Session-unique storage name. 
* `configPath` Server-side `*`.json path. 
* `embeddingModel` Optional catalog embedding-model name for cross-check; when non-empty must match the config's `embedding_model`. 
* `timeout` Maximum time to wait for the response; embedding large corpora is slow.



**Returns:**

[**EmbeddedStorageInfo**](struct_tryll_1_1_tryll_client_1_1_embedded_storage_info.md) with name, record count, and dimension.




**Exception:**


* `TryllError` On server-reported errors (including `InvalidStringStorageData` when `embeddingModel` mismatches the config) or timeout. 




        

<hr>



### function CreateEmbeddedStringStorageFromStrings 

_Create an_ `EmbeddedStringStorage` _from inline strings._
```C++
EmbeddedStorageInfo Tryll::TryllClient::CreateEmbeddedStringStorageFromStrings (
    std::string_view name,
    const std::vector< std::string > & strings,
    std::string_view embeddingModel,
    std::optional< std::chrono::milliseconds > timeout=std::nullopt
) 
```



Path B: the server embeds `strings` in memory with the named embedding model; no on-disk index is written.


If `allow_auto_model_downloading` was set in `ConfigureSession` and the required embedding model is absent, it will be downloaded before embedding begins. In that case the default timeout is automatically raised to 30 minutes; pass an explicit value to override.




**Parameters:**


* `name` Session-unique storage name. 
* `strings` Inline records to embed. 
* `embeddingModel` Catalog embedding-model name (required). 
* `timeout` Maximum time to wait; embedding is slow.



**Returns:**

[**EmbeddedStorageInfo**](struct_tryll_1_1_tryll_client_1_1_embedded_storage_info.md) with name, record count, and dimension.




**Exception:**


* `TryllError` On server-reported errors or timeout. 




        

<hr>



### function CreateStringStorage 

_Create a named_ `StringStorage` _on the server from an inline list._
```C++
void Tryll::TryllClient::CreateStringStorage (
    std::string_view name,
    const std::vector< std::string > & strings,
    std::chrono::milliseconds timeout=std::chrono::seconds{10}
) 
```



The storage may then be referenced by name in node params via `string_storage` (e.g. on `CannedResponseNode` or `HumanMessageGuardrailNode`). Must be called before [**CreateAgent**](class_tryll_1_1_tryll_client.md#function-createagent) on any graph that references this storage name.




**Parameters:**


* `name` Session-unique storage name. 
* `strings` Inline list of strings to store. 
* `timeout` Maximum time to wait for the response.



**Exception:**


* `TryllError` On server-reported errors or timeout. 




        

<hr>



### function CreateStringStorageFromFile 

_Create a named_ `StringStorage` _on the server from a server-side file._
```C++
void Tryll::TryllClient::CreateStringStorageFromFile (
    std::string_view name,
    std::string_view filePath,
    std::chrono::milliseconds timeout=std::chrono::seconds{10}
) 
```





**Parameters:**


* `name` Session-unique storage name. 
* `filePath` Server-side path to a newline-delimited text file. 
* `timeout` Maximum time to wait for the response.



**Exception:**


* `TryllError` On server-reported errors, missing file, or timeout. 




        

<hr>



### function DestroyEmbeddedStringStorage 

_Destroy a named_ `EmbeddedStringStorage` _._
```C++
void Tryll::TryllClient::DestroyEmbeddedStringStorage (
    std::string_view name,
    std::chrono::milliseconds timeout=std::chrono::seconds{10}
) 
```



Nodes that already hold the storage via `shared_ptr` keep it alive; this call only drops the session-level name mapping.




**Parameters:**


* `name` Name passed to one of the `Create` variants. 
* `timeout` Maximum time to wait for the response.



**Exception:**


* `TryllError` On server-reported errors or timeout. 




        

<hr>



### function DestroyStringStorage 

_Destroy a named_ `StringStorage` _on the server._
```C++
void Tryll::TryllClient::DestroyStringStorage (
    std::string_view name,
    std::chrono::milliseconds timeout=std::chrono::seconds{10}
) 
```



Nodes that already hold the storage via `shared_ptr` keep it alive; this call only drops the session-level name mapping.




**Parameters:**


* `name` Name passed to [**CreateStringStorage**](class_tryll_1_1_tryll_client.md#function-createstringstorage) or [**CreateStringStorageFromFile**](class_tryll_1_1_tryll_client.md#function-createstringstoragefromfile). 
* `timeout` Maximum time to wait for the response.



**Exception:**


* `TryllError` On server-reported errors or timeout. 




        

<hr>



### function DownloadModel 

_Start downloading a model on the server and block until complete._ 
```C++
void Tryll::TryllClient::DownloadModel (
    std::string_view modelName,
    Client::DownloadProgressCallback onProgress=nullptr,
    std::chrono::milliseconds timeout=std::chrono::minutes{30}
) 
```



Progress frames are delivered via `onProgress` on the reader thread (invoked synchronously, so keep the callback fast and non-blocking).




**Parameters:**


* `modelName` Catalog model name from `models.json`. 
* `onProgress` Optional per-chunk progress callback. 
* `timeout` Maximum time to wait for `DownloadComplete`.



**Exception:**


* `TryllError` On server-reported errors, a `DownloadComplete` reporting failure, or timeout. 




        

<hr>



### function GetSessionId 

_Server-assigned session identifier._ 
```C++
std::uint64_t Tryll::TryllClient::GetSessionId () noexcept const
```





**Returns:**

The `session_id` field received in `SessionReady`. 





        

<hr>



### function ListModels 

_Request the list of models known to the server for the session's engine._ 
```C++
std::vector< Client::ModelInfo > Tryll::TryllClient::ListModels (
    std::chrono::milliseconds timeout=std::chrono::seconds{10}
) 
```





**Parameters:**


* `timeout` Maximum time to wait for `ListModelsResponse`.



**Returns:**

Catalog entries, one per known model.




**Exception:**


* `TryllError` On server-reported errors or timeout. 




        

<hr>



### function LoadModel 

_Explicitly load and pin a model into memory._ 
```C++
void Tryll::TryllClient::LoadModel (
    std::string_view modelName,
    std::chrono::milliseconds timeout=std::chrono::minutes{5}
) 
```



The model stays in memory until [**UnloadModel**](class_tryll_1_1_tryll_client.md#function-unloadmodel) is called, regardless of whether any agents are using it.




**Parameters:**


* `modelName` Catalog model name from `models.json`. 
* `timeout` Maximum time to wait; loading is slow for large GGUFs.



**Exception:**


* `TryllError` If the model cannot be resolved or loaded, or on timeout. 




        

<hr>



### function Shutdown 

_Blocking convenience wrapper over_ [_**ShutdownAsync**_](class_tryll_1_1_tryll_client.md#function-shutdownasync) _._
```C++
void Tryll::TryllClient::Shutdown (
    std::chrono::milliseconds timeout=std::chrono::seconds{30}
) 
```





**Parameters:**


* `timeout` Maximum time to wait for a graceful close.



**Exception:**


* `TryllError` On timeout; socket errors during shutdown are suppressed. 




        

<hr>



### function ShutdownAsync 

_Asynchronously shut down the connection._ 
```C++
std::future< void > Tryll::TryllClient::ShutdownAsync () 
```



Pending requests are cancelled with [**TryllError**](class_tryll_1_1_tryll_error.md). Idempotent.




**Returns:**

Future completing when the socket is closed. 





        

<hr>



### function TryllClient [1/3]

```C++
Tryll::TryllClient::TryllClient (
    const TryllClient &
) = default
```




<hr>



### function TryllClient [2/3]

```C++
Tryll::TryllClient::TryllClient (
    TryllClient &&
) noexcept
```




<hr>



### function UnloadModel 

_Unpin a previously pinned model._ 
```C++
void Tryll::TryllClient::UnloadModel (
    std::string_view modelName,
    std::chrono::milliseconds timeout=std::chrono::seconds{30}
) 
```



Freed immediately if no active contexts reference it; freed lazily when the last one is destroyed.




**Parameters:**


* `modelName` Catalog model name previously pinned with [**LoadModel**](class_tryll_1_1_tryll_client.md#function-loadmodel). 
* `timeout` Maximum time to wait for the ack.



**Exception:**


* `TryllError` On timeout. 




        

<hr>



### function operator= 

```C++
TryllClient & Tryll::TryllClient::operator= (
    const TryllClient &
) = default
```




<hr>



### function operator= 

```C++
TryllClient & Tryll::TryllClient::operator= (
    TryllClient &&
) noexcept
```




<hr>



### function ~TryllClient 

```C++
Tryll::TryllClient::~TryllClient () 
```




<hr>
## Public Static Functions Documentation




### function Connect 

_Blocking convenience wrapper over_ [_**ConnectAsync**_](class_tryll_1_1_tryll_client.md#function-connectasync) _._
```C++
static TryllClient Tryll::TryllClient::Connect (
    std::string_view host,
    std::uint16_t port,
    std::chrono::milliseconds timeout=std::chrono::seconds{30}
) 
```





**Parameters:**


* `host` Hostname or IP of the Tryll server. 
* `port` TCP port. 
* `timeout` Maximum time to wait for `SessionReady`.



**Returns:**

A ready-to-use [**TryllClient**](class_tryll_1_1_tryll_client.md).




**Exception:**


* `TryllError` On connection failure, timeout, or a failed session handshake. 




        

<hr>



### function ConnectAsync 

_Asynchronously connect and wait for_ `SessionReady` _._
```C++
static std::future< TryllClient > Tryll::TryllClient::ConnectAsync (
    std::string_view host,
    std::uint16_t port
) 
```





**Parameters:**


* `host` Hostname or IP of the Tryll server. 
* `port` TCP port the server is listening on.



**Returns:**

Future yielding a ready-to-use [**TryllClient**](class_tryll_1_1_tryll_client.md). Calling `future::get()` propagates any [**TryllError**](class_tryll_1_1_tryll_error.md) raised during the handshake. 





        

<hr>



### function RunAndConnect 

_Spawn a local server and connect to it — recommended for single-process deployments where the client owns the server lifecycle._ 
```C++
static ConnectedSession Tryll::TryllClient::RunAndConnect (
    Client::ManagedServerOptions serverOpts,
    std::chrono::milliseconds connectTimeout=std::chrono::seconds{30}
) 
```



Equivalent to calling [**Client::ManagedServer::Start**](class_tryll_1_1_client_1_1_managed_server.md#function-start) followed by [**Connect**](class_tryll_1_1_tryll_client.md#function-connect), with correct cleanup if `Connect` throws. The returned [**ConnectedSession**](class_tryll_1_1_connected_session.md) owns both the server process and the TCP session; on destruction it shuts the client down first, then terminates the server.




**Parameters:**


* `serverOpts` Server launch options. `serverOpts.exe` is required. 
* `connectTimeout` Maximum time to wait for `SessionReady` after the server's TCP port becomes reachable.



**Returns:**

A [**ConnectedSession**](class_tryll_1_1_connected_session.md) ready to use.




**Exception:**


* `TryllError` If the server cannot be started, the TCP ready-probe times out, or the session handshake fails. 




        

<hr>



### function RunAndConnectAsync 

_Asynchronous variant of_ [_**RunAndConnect**_](class_tryll_1_1_tryll_client.md#function-runandconnect) _._
```C++
static std::future< ConnectedSession > Tryll::TryllClient::RunAndConnectAsync (
    Client::ManagedServerOptions serverOpts
) 
```



Spawns a `std::async` task that calls [**RunAndConnect**](class_tryll_1_1_tryll_client.md#function-runandconnect) internally. Useful when the calling thread must not block during startup.




**Parameters:**


* `serverOpts` Server launch options.



**Returns:**

Future yielding a [**ConnectedSession**](class_tryll_1_1_connected_session.md). Calling `future::get()` propagates any [**TryllError**](class_tryll_1_1_tryll_error.md) from the launch or handshake. 





        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo/server/client-cpp/include/tryll/TryllClient.h`

