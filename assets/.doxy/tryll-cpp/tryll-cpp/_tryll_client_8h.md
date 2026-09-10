

# File TryllClient.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**cpp**](dir_bfb9b1426fa54ff893373dfbed66c670.md) **>** [**include**](dir_3428be636e548afd883ca44bef078c76.md) **>** [**tryll**](dir_3ee824def4798d48f3bfea3e4f65c1f8.md) **>** [**TryllClient.h**](_tryll_client_8h.md)



_TCP session to the Tryll server and its supporting value types._ [More...](#detailed-description)

* `#include <tryll/AgentCreateOptions.h>`
* `#include <tryll/AgentProxy.h>`
* `#include <tryll/GraphDescription.h>`
* `#include <tryll/ManagedServer.h>`
* `#include <tryll/TryllError.h>`
* `#include <tryll/VoiceInput.h>`
* `#include <messages_generated.h>`
* `#include <chrono>`
* `#include <cstdint>`
* `#include <functional>`
* `#include <future>`
* `#include <memory>`
* `#include <optional>`
* `#include <string>`
* `#include <string_view>`
* `#include <vector>`













## Namespaces

| Type | Name |
| ---: | :--- |
| namespace | [**Tryll**](namespace_tryll.md) <br> |
| namespace | [**Client**](namespace_tryll_1_1_client.md) <br> |
| namespace | [**Internal**](namespace_tryll_1_1_client_1_1_internal.md) <br> |


## Classes

| Type | Name |
| ---: | :--- |
| class | [**ConnectedSession**](class_tryll_1_1_client_1_1_connected_session.md) <br>_RAII pair: owns a_ [_**ManagedServer**_](class_tryll_1_1_client_1_1_managed_server.md) _child process and a connected_[_**TryllClient**_](class_tryll_1_1_client_1_1_tryll_client.md) _._ |
| class | [**TryllClient**](class_tryll_1_1_client_1_1_tryll_client.md) <br>_TCP session to the Tryll server._  |
| struct | [**DebugCommandResult**](struct_tryll_1_1_client_1_1_tryll_client_1_1_debug_command_result.md) <br> |
| struct | [**EmbeddedStorageInfo**](struct_tryll_1_1_client_1_1_tryll_client_1_1_embedded_storage_info.md) <br>_Result returned by_ `CreateEmbeddedStringStorage` _variants._ |
| struct | [**EngineSelection**](struct_tryll_1_1_client_1_1_tryll_client_1_1_engine_selection.md) <br> |
| struct | [**LoadModelResult**](struct_tryll_1_1_client_1_1_tryll_client_1_1_load_model_result.md) <br>_Explicitly load and pin a model into memory._  |
| struct | [**ModelPlacementHint**](struct_tryll_1_1_client_1_1_tryll_client_1_1_model_placement_hint.md) <br> |
| struct | [**SessionConfig**](struct_tryll_1_1_client_1_1_tryll_client_1_1_session_config.md) <br> |


















































## Detailed Description


Declares [**Tryll::Client::TryllClient**](class_tryll_1_1_client_1_1_tryll_client.md), the primary client class, and [**Tryll::Client::ConnectedSession**](class_tryll_1_1_client_1_1_connected_session.md), the recommended one-call entry point for local-server deployments. Model-catalog entries are returned as `::Tryll::ModelInfoT` (FlatBuffers gen-object-api type).


Recommended flow (local server, automatic lifecycle):



```C++
Tryll::Client::ManagedServerOptions opts;
opts.exe  = "path/to/tryll_server.exe";
opts.port = 9100;
auto session = Tryll::Client::TryllClient::RunAndConnect(opts);
// An empty config is the normal case: each model's engine comes from the
// server's catalog. Name engines only to pre-warm them (see SessionConfig).
session.GetClient().CreateSession({});
auto agent = session.GetClient().CreateAgent(graph);
agent.SendText("hi");
agent.Destroy();
// session destructor: shuts down client, then stops the server
```



Advanced / remote-server flow (no server management):



```C++
auto client = Tryll::Client::TryllClient::Connect("127.0.0.1", 9100);
client.CreateSession({});
auto agent = client.CreateAgent(graph);
agent.SendText("hi");
agent.Destroy();
client.Shutdown();
```
 


    

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/cpp/include/tryll/TryllClient.h`

