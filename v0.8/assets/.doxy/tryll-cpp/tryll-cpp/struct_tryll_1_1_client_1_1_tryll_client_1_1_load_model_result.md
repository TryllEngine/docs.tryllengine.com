

# Struct Tryll::Client::TryllClient::LoadModelResult



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**TryllClient**](class_tryll_1_1_client_1_1_tryll_client.md) **>** [**LoadModelResult**](struct_tryll_1_1_client_1_1_tryll_client_1_1_load_model_result.md)



_Explicitly load and pin a model into memory._ [More...](#detailed-description)

* `#include <TryllClient.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::string | [**backend**](#variable-backend)  <br> |
|  ::Tryll::MemoryPlacement | [**placement**](#variable-placement)   = `::Tryll::MemoryPlacement\_Auto`<br> |












































## Detailed Description


The model stays in memory until [**UnloadModel**](class_tryll_1_1_client_1_1_tryll_client.md#function-unloadmodel) is called, regardless of whether any agents are using it.




**Parameters:**


* `modelName` Catalog model name from `models.json`. 
* `timeout` Maximum time to wait; loading is slow for large GGUFs.



**Exception:**


* `TryllError` If the model cannot be resolved or loaded, or on timeout. What a [**LoadModel**](class_tryll_1_1_client_1_1_tryll_client.md#function-loadmodel) actually did. 




    
## Public Attributes Documentation




### variable backend 

```C++
std::string Tryll::Client::TryllClient::LoadModelResult::backend;
```



Backend that served the load ("vulkan" \| "cuda" \| "rocm" \| "cpu", or the ONNX Runtime provider for Sherpa). Read-only: a client never picks this. Empty when the load resolved no backend. 


        

<hr>



### variable placement 

```C++
::Tryll::MemoryPlacement Tryll::Client::TryllClient::LoadModelResult::placement;
```



Where the model ended up. Never `Auto`, and **not** an echo of the request: an unsatisfiable `Vram` reports `Ram`, and a model already resident under a different placement reports the resident value — placement is fixed at load time and is deliberately not part of the server's model cache key, so the first loader wins. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/TryllClient.h`

