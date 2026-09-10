

# Struct Tryll::Client::TryllClient::ModelPlacementHint



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**TryllClient**](class_tryll_1_1_client_1_1_tryll_client.md) **>** [**ModelPlacementHint**](struct_tryll_1_1_client_1_1_tryll_client_1_1_model_placement_hint.md)



[More...](#detailed-description)

* `#include <TryllClient.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  ::Tryll::HotwordBias | [**hotwords**](#variable-hotwords)   = `::Tryll::HotwordBias\_Default`<br> |
|  std::string | [**modelName**](#variable-modelname)  <br> |
|  ::Tryll::MemoryPlacement | [**placement**](#variable-placement)   = `::Tryll::MemoryPlacement\_Auto`<br> |












































## Detailed Description


A per-model hint. Sparse: list only the models you have an opinion about; anything absent takes every field's default.


`placement` is a hint, never a command: a host that cannot honour it downgrades and reports what it did — read `LoadModelResponse` rather than assuming.


`hotwords` is _not_ downgradable. Hotword biasing needs beam-search decoding, which a recognizer fixes when it is constructed, so it selects which recognizer this session is served. `HotwordBias_On` costs a few percent of decode time and changes that model's transcripts whether or not a lexicon is supplied; `HotwordBias_Off` makes `CreateVoiceInput` reject a `hotwordsStoragePath` with `HotwordBiasDisabled` (4106) instead of accepting it and biasing nothing. Left at `HotwordBias_Default`, the model's catalog entry decides. 


    
## Public Attributes Documentation




### variable hotwords 

```C++
::Tryll::HotwordBias Tryll::Client::TryllClient::ModelPlacementHint::hotwords;
```




<hr>



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
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/TryllClient.h`

