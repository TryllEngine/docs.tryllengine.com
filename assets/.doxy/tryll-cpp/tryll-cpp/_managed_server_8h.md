

# File ManagedServer.h



[**FileList**](files.md) **>** [**client-cpp**](dir_609af48e6c91725f4b3bc7ea6c8d260d.md) **>** [**include**](dir_e007c6614a1ec85f1d1448223783a2c8.md) **>** [**tryll**](dir_8463b878364fc27e5e1890816f6531c6.md) **>** [**ManagedServer.h**](_managed_server_8h.md)



_RAII wrapper that spawns a_ `tryll_server` _child process and waits for it to become reachable on its TCP port._[More...](#detailed-description)

* `#include <tryll/TryllError.h>`
* `#include <chrono>`
* `#include <cstdint>`
* `#include <filesystem>`
* `#include <memory>`
* `#include <string>`
* `#include <vector>`













## Namespaces

| Type | Name |
| ---: | :--- |
| namespace | [**Tryll**](namespace_tryll.md) <br> |
| namespace | [**Client**](namespace_tryll_1_1_client.md) <br> |


## Classes

| Type | Name |
| ---: | :--- |
| class | [**ManagedServer**](class_tryll_1_1_client_1_1_managed_server.md) <br>_RAII handle around a child_ `tryll_server` _process._ |
| struct | [**ManagedServerOptions**](struct_tryll_1_1_client_1_1_managed_server_options.md) <br>_Configuration passed to_ [_**ManagedServer::Start**_](class_tryll_1_1_client_1_1_managed_server.md#function-start) _._ |


















































## Detailed Description


Typical usage:



```C++
Tryll::Client::ManagedServerOptions opts;
opts.exe  = "C:/tryll/tryll_server.exe";
opts.port = 9100;

auto server = Tryll::Client::ManagedServer::Start(opts);
auto client = Tryll::TryllClient::Connect(server.Host(), server.Port());
// … use client …
client.Shutdown();
server.Stop();   // or just let the destructor run
```





**Note:**

Windows-only. A `#error` guards non-Windows builds. 





    

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-cpp/include/tryll/ManagedServer.h`

