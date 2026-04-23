

# Struct Tryll::Client::RetrievePresentationConfig



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**RetrievePresentationConfig**](struct_tryll_1_1_client_1_1_retrieve_presentation_config.md)



_Formatting rules for one knowledge source (or the default for all sources)._ [More...](#detailed-description)

* `#include <GraphDescription.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::string | [**blockPrefix**](#variable-blockprefix)  <br>_String prepended to the whole block of chunks for this source._  |
|  std::string | [**blockSuffix**](#variable-blocksuffix)  <br>_String appended to the whole block of chunks for this source._  |
|  std::string | [**chunkPrefix**](#variable-chunkprefix)  <br>_String prepended to each retrieved chunk._  |
|  std::string | [**chunkSuffix**](#variable-chunksuffix)  <br>_String appended to each retrieved chunk._  |
|  bool | [**emitWhenEmpty**](#variable-emitwhenempty)   = `false`<br>_If true, emit_ `emptyText` _even when this retriever returned nothing._ |
|  std::string | [**emptyText**](#variable-emptytext)  <br>_Text emitted when this retriever is empty and_ `emitWhenEmpty` _is true._ |
|  std::string | [**source**](#variable-source)  <br>_Label key matching a_ `RetrieveNode` __`source` _; empty = default config._ |












































## Detailed Description


Used both as the default entry on [**KnowledgePresentationConfig**](struct_tryll_1_1_client_1_1_knowledge_presentation_config.md) and, keyed by `source`, as per-source overrides. 


    
## Public Attributes Documentation




### variable blockPrefix 

_String prepended to the whole block of chunks for this source._ 
```C++
std::string Tryll::Client::RetrievePresentationConfig::blockPrefix;
```




<hr>



### variable blockSuffix 

_String appended to the whole block of chunks for this source._ 
```C++
std::string Tryll::Client::RetrievePresentationConfig::blockSuffix;
```




<hr>



### variable chunkPrefix 

_String prepended to each retrieved chunk._ 
```C++
std::string Tryll::Client::RetrievePresentationConfig::chunkPrefix;
```




<hr>



### variable chunkSuffix 

_String appended to each retrieved chunk._ 
```C++
std::string Tryll::Client::RetrievePresentationConfig::chunkSuffix;
```




<hr>



### variable emitWhenEmpty 

_If true, emit_ `emptyText` _even when this retriever returned nothing._
```C++
bool Tryll::Client::RetrievePresentationConfig::emitWhenEmpty;
```




<hr>



### variable emptyText 

_Text emitted when this retriever is empty and_ `emitWhenEmpty` _is true._
```C++
std::string Tryll::Client::RetrievePresentationConfig::emptyText;
```




<hr>



### variable source 

_Label key matching a_ `RetrieveNode` __`source` _; empty = default config._
```C++
std::string Tryll::Client::RetrievePresentationConfig::source;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/tryll-mono/server/client-cpp/include/tryll/GraphDescription.h`

