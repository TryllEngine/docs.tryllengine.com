

# Struct Tryll::Client::ModelInfo



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**ModelInfo**](struct_tryll_1_1_client_1_1_model_info.md)



_Summary information for one model returned by_ [_**Tryll::TryllClient::ListModels**_](class_tryll_1_1_tryll_client.md#function-listmodels) _._

* `#include <TryllClient.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::string | [**hfRepo**](#variable-hfrepo)  <br>_HuggingFace repo id, or empty for local-only models._  |
|  std::string | [**name**](#variable-name)  <br>_Catalog name of the model (key in_ `models.json` _)._ |
|  std::uint64\_t | [**sizeBytes**](#variable-sizebytes)   = `0`<br>_Total on-disk size in bytes; 0 for_ `Absent` _models._ |
|  [**ModelStatus**](namespace_tryll_1_1_client.md#enum-modelstatus) | [**status**](#variable-status)   = `ModelStatus::Absent`<br>_Current status; see_ [_**ModelStatus**_](namespace_tryll_1_1_client.md#enum-modelstatus) _._ |












































## Public Attributes Documentation




### variable hfRepo 

_HuggingFace repo id, or empty for local-only models._ 
```C++
std::string Tryll::Client::ModelInfo::hfRepo;
```




<hr>



### variable name 

_Catalog name of the model (key in_ `models.json` _)._
```C++
std::string Tryll::Client::ModelInfo::name;
```




<hr>



### variable sizeBytes 

_Total on-disk size in bytes; 0 for_ `Absent` _models._
```C++
std::uint64_t Tryll::Client::ModelInfo::sizeBytes;
```




<hr>



### variable status 

_Current status; see_ [_**ModelStatus**_](namespace_tryll_1_1_client.md#enum-modelstatus) _._
```C++
ModelStatus Tryll::Client::ModelInfo::status;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/tryll-mono/server/client-cpp/include/tryll/TryllClient.h`

