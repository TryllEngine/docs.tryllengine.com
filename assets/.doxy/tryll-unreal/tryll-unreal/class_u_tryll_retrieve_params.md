

# Class UTryllRetrieveParams



[**ClassList**](annotated.md) **>** [**UTryllRetrieveParams**](class_u_tryll_retrieve_params.md)



[More...](#detailed-description)

* `#include <TryllRetrieveParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  [**FTryllStoragePath**](struct_f_tryll_storage_path.md) | [**EmbeddedStringStorage**](#variable-embeddedstringstorage)  <br> |
|  FString | [**Filter**](#variable-filter)  <br> |
|  FString | [**FoundExit**](#variable-foundexit)  <br> |
|  FString | [**NotFoundExit**](#variable-notfoundexit)  <br> |
|  FString | [**Source**](#variable-source)  <br> |
|  float | [**Threshold**](#variable-threshold)   = `0.5f`<br> |
|  int32 | [**TopK**](#variable-topk)   = `2`<br> |
|  bool | [**bOverrideFilter**](#variable-boverridefilter)   = `false`<br> |
|  bool | [**bOverrideSource**](#variable-boverridesource)   = `false`<br> |
































## Public Functions

| Type | Name |
| ---: | :--- |
| virtual ETryllNodeType | [**GetNodeType**](#function-getnodetype) () override const<br> |
| virtual flatbuffers::Offset&lt; void &gt; | [**Pack**](#function-pack) (flatbuffers::FlatBufferBuilder & Fbb, Tryll::NodeParams::NodeParams & OutType) override const<br> |


## Public Functions inherited from UTryllNodeParamsBase

See [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)

| Type | Name |
| ---: | :--- |
| virtual TArray&lt; FString &gt; | [**GetExitTargetOptions**](class_u_tryll_node_params_base.md#function-getexittargetoptions) () const<br> |
| virtual ETryllNodeType | [**GetNodeType**](class_u_tryll_node_params_base.md#function-getnodetype) () const<br> |
| virtual flatbuffers::Offset&lt; void &gt; | [**Pack**](class_u_tryll_node_params_base.md#function-pack) (flatbuffers::FlatBufferBuilder & Fbb, ::Tryll::NodeParams::NodeParams & OutType) const<br> |
|  ETryllNodeType return | [**static\_cast**](class_u_tryll_node_params_base.md#function-static_cast) (0) <br> |






















































## Detailed Description


RAG retrieval node. Embeds the HumanMessage, searches an embedded storage, and attaches a KnowledgeComponent to the current interaction. 


    
## Public Attributes Documentation




### variable EmbeddedStringStorage 

```C++
FTryllStoragePath UTryllRetrieveParams::EmbeddedStringStorage;
```



Named embedded string storage (EmbeddedStringStorageManager). Structural because the storage is resolved and referenced at construction. 


        

<hr>



### variable Filter 

```C++
FString UTryllRetrieveParams::Filter;
```




<hr>



### variable FoundExit 

```C++
FString UTryllRetrieveParams::FoundExit;
```



Exit taken when retrieval returned at least one chunk above threshold. Empty string = END. Graph exit "found" — target node name; empty = END. 


        

<hr>



### variable NotFoundExit 

```C++
FString UTryllRetrieveParams::NotFoundExit;
```



Exit taken when retrieval produced no results. Empty string = END. Graph exit "not\_found" — target node name; empty = END. 


        

<hr>



### variable Source 

```C++
FString UTryllRetrieveParams::Source;
```




<hr>



### variable Threshold 

```C++
float UTryllRetrieveParams::Threshold;
```



Maximum cosine distance; results above this threshold are dropped. Zero disables filtering (no threshold). 


        

<hr>



### variable TopK 

```C++
int32 UTryllRetrieveParams::TopK;
```



Number of chunks to retrieve. 


        

<hr>



### variable bOverrideFilter 

```C++
bool UTryllRetrieveParams::bOverrideFilter;
```



JSON filter compiled against the storage's metadata schema. Empty string = no filter. Mutable — recompiles the filter on change. Set to true to override the inherited Filter value. 


        

<hr>



### variable bOverrideSource 

```C++
bool UTryllRetrieveParams::bOverrideSource;
```



Label attached to the KnowledgeComponent; defaults to node name. Set to true to override the inherited Source value. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllRetrieveParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllRetrieveParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllRetrieveParams.h`

