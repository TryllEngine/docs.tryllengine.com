

# File ManagedServer.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**cpp**](dir_bfb9b1426fa54ff893373dfbed66c670.md) **>** [**include**](dir_3428be636e548afd883ca44bef078c76.md) **>** [**tryll**](dir_3ee824def4798d48f3bfea3e4f65c1f8.md) **>** [**ManagedServer.h**](_managed_server_8h.md)



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
auto client = Tryll::Client::TryllClient::Connect(server.Host(), server.Port());
// … use client …
client.Shutdown();
server.Stop();   // or just let the destructor run
```





**Note:**

Windows-only. A `#error` guards non-Windows builds. 





    

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/ManagedServer.h`

