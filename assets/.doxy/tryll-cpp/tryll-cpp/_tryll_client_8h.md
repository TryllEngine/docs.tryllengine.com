

# File TryllClient.h



[**FileList**](files.md) **>** [**client-cpp**](dir_609af48e6c91725f4b3bc7ea6c8d260d.md) **>** [**include**](dir_e007c6614a1ec85f1d1448223783a2c8.md) **>** [**tryll**](dir_8463b878364fc27e5e1890816f6531c6.md) **>** [**TryllClient.h**](_tryll_client_8h.md)



_TCP session to the Tryll server and its supporting value types._ [More...](#detailed-description)

* `#include <tryll/AgentProxy.h>`
* `#include <tryll/GraphDescription.h>`
* `#include <tryll/TryllError.h>`
* `#include <chrono>`
* `#include <cstdint>`
* `#include <functional>`
* `#include <future>`
* `#include <memory>`
* `#include <string>`
* `#include <string_view>`
* `#include <vector>`













## Namespaces

| Type | Name |
| ---: | :--- |
| namespace | [**Tryll**](namespace_tryll.md) <br> |
| namespace | [**Client**](namespace_tryll_1_1_client.md) <br> |


## Classes

| Type | Name |
| ---: | :--- |
| struct | [**ModelInfo**](struct_tryll_1_1_client_1_1_model_info.md) <br>_Summary information for one model returned by_ [_**Tryll::TryllClient::ListModels**_](class_tryll_1_1_tryll_client.md#function-listmodels) _._ |
| class | [**TryllClient**](class_tryll_1_1_tryll_client.md) <br>_TCP session to the Tryll server._  |
| struct | [**EmbeddedStorageInfo**](struct_tryll_1_1_tryll_client_1_1_embedded_storage_info.md) <br>_Result returned by_ `CreateEmbeddedStringStorage` _variants._ |


















































## Detailed Description


Declares [**Tryll::TryllClient**](class_tryll_1_1_tryll_client.md), the entry point for the C++ client library. The session-level types — [**Tryll::Client::ModelStatus**](namespace_tryll_1_1_client.md#enum-modelstatus), [**Tryll::Client::ModelInfo**](struct_tryll_1_1_client_1_1_model_info.md), [**Tryll::Client::DownloadProgressCallback**](namespace_tryll_1_1_client.md#typedef-downloadprogresscallback) — live in the `Tryll::Client` namespace.


Typical connect-create-use flow:



```C++
auto client = Tryll::TryllClient::Connect("127.0.0.1", 9100);
client.ConfigureSession(Tryll::Client::InferenceEngine::LlamaCpp);
auto agent = client.CreateAgent(graph);
agent.SendText("hi");
agent.Destroy();
client.Shutdown();
```
 


    

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/tryll-mono/server/client-cpp/include/tryll/TryllClient.h`

