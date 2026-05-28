

# Struct Tryll::TryllClient::EmbeddedStorageInfo



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**TryllClient**](class_tryll_1_1_tryll_client.md) **>** [**EmbeddedStorageInfo**](struct_tryll_1_1_tryll_client_1_1_embedded_storage_info.md)



_Result returned by_ `CreateEmbeddedStringStorage` _variants._

* `#include <TryllClient.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::uint32\_t | [**embeddingDim**](#variable-embeddingdim)   = `0`<br>_Embedding vector dimensionality._  |
|  std::string | [**name**](#variable-name)  <br>_Storage name as registered on the server._  |
|  std::uint32\_t | [**recordCount**](#variable-recordcount)   = `0`<br>_Number of records indexed in the storage._  |












































## Public Attributes Documentation




### variable embeddingDim 

_Embedding vector dimensionality._ 
```C++
std::uint32_t Tryll::TryllClient::EmbeddedStorageInfo::embeddingDim;
```




<hr>



### variable name 

_Storage name as registered on the server._ 
```C++
std::string Tryll::TryllClient::EmbeddedStorageInfo::name;
```




<hr>



### variable recordCount 

_Number of records indexed in the storage._ 
```C++
std::uint32_t Tryll::TryllClient::EmbeddedStorageInfo::recordCount;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-cpp/include/tryll/TryllClient.h`

