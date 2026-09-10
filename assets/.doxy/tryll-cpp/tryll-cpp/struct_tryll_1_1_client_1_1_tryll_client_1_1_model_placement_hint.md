

# Struct Tryll::Client::TryllClient::ModelPlacementHint



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**TryllClient**](class_tryll_1_1_client_1_1_tryll_client.md) **>** [**ModelPlacementHint**](struct_tryll_1_1_client_1_1_tryll_client_1_1_model_placement_hint.md)



[More...](#detailed-description)

* `#include <TryllClient.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::string | [**modelName**](#variable-modelname)  <br> |
|  ::Tryll::MemoryPlacement | [**placement**](#variable-placement)   = `::Tryll::MemoryPlacement\_Auto`<br> |












































## Detailed Description


A per-model placement hint. Sparse: list only the models you have an opinion about; anything absent means `MemoryPlacement_Auto`.


A hint, never a command. A host that cannot honour it downgrades and reports what it did — read `LoadModelResponse` rather than assuming. 


    
## Public Attributes Documentation




### variable modelName 

```C++
std::string Tryll::Client::TryllClient::ModelPlacementHint::modelName;
```




<hr>



### variable placement 

```C++
::Tryll::MemoryPlacement Tryll::Client::TryllClient::ModelPlacementHint::placement;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/cpp/include/tryll/TryllClient.h`

