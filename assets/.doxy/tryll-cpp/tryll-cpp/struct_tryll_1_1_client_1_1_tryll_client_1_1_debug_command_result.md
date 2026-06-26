

# Struct Tryll::Client::TryllClient::DebugCommandResult



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**TryllClient**](class_tryll_1_1_client_1_1_tryll_client.md) **>** [**DebugCommandResult**](struct_tryll_1_1_client_1_1_tryll_client_1_1_debug_command_result.md)



[More...](#detailed-description)

* `#include <TryllClient.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  bool | [**acknowledged**](#variable-acknowledged)   = `false`<br>_Server replied with DebugCommandResponse (non-fatal verb)._  |
|  bool | [**disconnected**](#variable-disconnected)   = `false`<br>_Server dropped the connection (a fatal verb like_ `crash` _succeeded)._ |
|  std::uint32\_t | [**errorCode**](#variable-errorcode)   = `0`<br>_Non-zero → server returned an ErrorResponse (e.g. 5005 = debug commands unavailable)._  |
|  std::string | [**message**](#variable-message)  <br>_Error message, or the response_ `result` _string._ |












































## Detailed Description


Outcome of [**SendDebugCommand**](class_tryll_1_1_client_1_1_tryll_client.md#function-senddebugcommand). Exactly one of the first three flags is set on a definite result; all false means the call timed out (no reply, no disconnect — the server is presumably still alive). 


    
## Public Attributes Documentation




### variable acknowledged 

_Server replied with DebugCommandResponse (non-fatal verb)._ 
```C++
bool Tryll::Client::TryllClient::DebugCommandResult::acknowledged;
```




<hr>



### variable disconnected 

_Server dropped the connection (a fatal verb like_ `crash` _succeeded)._
```C++
bool Tryll::Client::TryllClient::DebugCommandResult::disconnected;
```




<hr>



### variable errorCode 

_Non-zero → server returned an ErrorResponse (e.g. 5005 = debug commands unavailable)._ 
```C++
std::uint32_t Tryll::Client::TryllClient::DebugCommandResult::errorCode;
```




<hr>



### variable message 

_Error message, or the response_ `result` _string._
```C++
std::string Tryll::Client::TryllClient::DebugCommandResult::message;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/TryllClient.h`

