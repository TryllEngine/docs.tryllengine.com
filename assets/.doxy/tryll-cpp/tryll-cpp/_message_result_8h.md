

# File MessageResult.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**cpp**](dir_bfb9b1426fa54ff893373dfbed66c670.md) **>** [**include**](dir_3428be636e548afd883ca44bef078c76.md) **>** [**tryll**](dir_3ee824def4798d48f3bfea3e4f65c1f8.md) **>** [**MessageResult.h**](_message_result_8h.md)



_Placeholder return type for per-turn metadata._ [More...](#detailed-description)














## Namespaces

| Type | Name |
| ---: | :--- |
| namespace | [**Tryll**](namespace_tryll.md) <br> |
| namespace | [**Client**](namespace_tryll_1_1_client.md) <br> |


## Classes

| Type | Name |
| ---: | :--- |
| struct | [**MessageResult**](struct_tryll_1_1_client_1_1_message_result.md) <br>_Placeholder for per-turn metadata returned from SendText._  |


















































## Detailed Description


Reserved for future result metadata such as token counts, latency, and diagnostics snapshots. In the current API [**Tryll::Client::AgentProxy::SendText**](class_tryll_1_1_client_1_1_agent_proxy.md#function-sendtext) returns `void`; this header exists for forward compatibility so callers can adopt [**Tryll::Client::MessageResult**](struct_tryll_1_1_client_1_1_message_result.md) as it grows. 


    

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/MessageResult.h`

