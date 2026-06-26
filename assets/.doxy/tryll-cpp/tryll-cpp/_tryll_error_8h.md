

# File TryllError.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**cpp**](dir_bfb9b1426fa54ff893373dfbed66c670.md) **>** [**include**](dir_3428be636e548afd883ca44bef078c76.md) **>** [**tryll**](dir_3ee824def4798d48f3bfea3e4f65c1f8.md) **>** [**TryllError.h**](_tryll_error_8h.md)



_Exception type raised by the Tryll C++ client library._ [More...](#detailed-description)

* `#include <stdexcept>`
* `#include <cstdint>`
* `#include <string>`













## Namespaces

| Type | Name |
| ---: | :--- |
| namespace | [**Tryll**](namespace_tryll.md) <br> |
| namespace | [**Client**](namespace_tryll_1_1_client.md) <br> |


## Classes

| Type | Name |
| ---: | :--- |
| class | [**TryllError**](class_tryll_1_1_client_1_1_tryll_error.md) <br>_Exception thrown by_ [_**TryllClient**_](class_tryll_1_1_client_1_1_tryll_client.md) _and_[_**AgentProxy**_](class_tryll_1_1_client_1_1_agent_proxy.md) _on server errors or disconnects._ |


















































## Detailed Description


All failures surfaced by [**Tryll::Client::TryllClient**](class_tryll_1_1_client_1_1_tryll_client.md) and [**Tryll::Client::AgentProxy**](class_tryll_1_1_client_1_1_agent_proxy.md) — server-reported errors, protocol decode failures, and connection-level problems — are raised as [**Tryll::Client::TryllError**](class_tryll_1_1_client_1_1_tryll_error.md). Server-reported errors carry a numeric code that matches the ranges defined in `server/common/include/tryll/ErrorCodes.h`; client-side errors (timeouts, closed socket) carry code 0. 


    

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/TryllError.h`

