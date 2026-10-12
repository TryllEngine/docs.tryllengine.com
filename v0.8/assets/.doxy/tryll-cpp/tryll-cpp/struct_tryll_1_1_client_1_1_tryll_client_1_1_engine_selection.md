

# Struct Tryll::Client::TryllClient::EngineSelection



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**TryllClient**](class_tryll_1_1_client_1_1_tryll_client.md) **>** [**EngineSelection**](struct_tryll_1_1_client_1_1_tryll_client_1_1_engine_selection.md)



[More...](#detailed-description)

* `#include <TryllClient.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  ::Tryll::InferenceEngine | [**engine**](#variable-engine)  <br> |
|  ::Tryll::ModelRole | [**role**](#variable-role)  <br> |












































## Detailed Description


One engine this session expects to touch, qualified by the role it serves.


Replaces the four per-role engine fields [**SessionConfig**](struct_tryll_1_1_client_1_1_tryll_client_1_1_session_config.md) carried through protocol v12. The server's catalog states each model's engine, so nothing here selects one — an entry only says "expect to use this", which lets the server initialise that engine's backend at CreateSession instead of at first load.


The role is redundant for lookup and survives so a pre-warm log line reads ("warmed llama-cpp for role language"). It costs one byte on the wire. One role may name several engines, and naming none is normal. 


    
## Public Attributes Documentation




### variable engine 

```C++
::Tryll::InferenceEngine Tryll::Client::TryllClient::EngineSelection::engine;
```




<hr>



### variable role 

```C++
::Tryll::ModelRole Tryll::Client::TryllClient::EngineSelection::role;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/cpp/include/tryll/TryllClient.h`

