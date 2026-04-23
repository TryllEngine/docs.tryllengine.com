

# File TryllError.h



[**FileList**](files.md) **>** [**client-cpp**](dir_609af48e6c91725f4b3bc7ea6c8d260d.md) **>** [**include**](dir_e007c6614a1ec85f1d1448223783a2c8.md) **>** [**tryll**](dir_8463b878364fc27e5e1890816f6531c6.md) **>** [**TryllError.h**](_tryll_error_8h.md)



_Exception type raised by the Tryll C++ client library._ [More...](#detailed-description)

* `#include <stdexcept>`
* `#include <cstdint>`
* `#include <string>`













## Namespaces

| Type | Name |
| ---: | :--- |
| namespace | [**Tryll**](namespace_tryll.md) <br> |


## Classes

| Type | Name |
| ---: | :--- |
| class | [**TryllError**](class_tryll_1_1_tryll_error.md) <br>_Exception thrown by_ [_**TryllClient**_](class_tryll_1_1_tryll_client.md) _and_[_**AgentProxy**_](class_tryll_1_1_agent_proxy.md) _on server errors or disconnects._ |


















































## Detailed Description


All failures surfaced by [**Tryll::TryllClient**](class_tryll_1_1_tryll_client.md) and [**Tryll::AgentProxy**](class_tryll_1_1_agent_proxy.md) — server-reported errors, protocol decode failures, and connection-level problems — are raised as [**Tryll::TryllError**](class_tryll_1_1_tryll_error.md). Server-reported errors carry a numeric code that matches the ranges defined in `server/common/include/tryll/ErrorCodes.h`; client-side errors (timeouts, closed socket) carry code 0. 


    

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/tryll-mono/server/client-cpp/include/tryll/TryllError.h`

