

# File MessageResult.h



[**FileList**](files.md) **>** [**client-cpp**](dir_609af48e6c91725f4b3bc7ea6c8d260d.md) **>** [**include**](dir_e007c6614a1ec85f1d1448223783a2c8.md) **>** [**tryll**](dir_8463b878364fc27e5e1890816f6531c6.md) **>** [**MessageResult.h**](_message_result_8h.md)



_Placeholder return type for per-turn metadata._ [More...](#detailed-description)














## Namespaces

| Type | Name |
| ---: | :--- |
| namespace | [**Tryll**](namespace_tryll.md) <br> |


## Classes

| Type | Name |
| ---: | :--- |
| struct | [**MessageResult**](struct_tryll_1_1_message_result.md) <br>_Placeholder for per-turn metadata returned from SendText._  |


















































## Detailed Description


Reserved for future result metadata such as token counts, latency, and diagnostics snapshots. In the current API [**Tryll::AgentProxy::SendText**](class_tryll_1_1_agent_proxy.md#function-sendtext) returns `void`; this header exists for forward compatibility so callers can adopt [**Tryll::MessageResult**](struct_tryll_1_1_message_result.md) as it grows. 


    

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo/server/client-cpp/include/tryll/MessageResult.h`

