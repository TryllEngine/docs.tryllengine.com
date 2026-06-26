

# Class UTryllClassifyIntentParams



[**ClassList**](annotated.md) **>** [**UTryllClassifyIntentParams**](class_u_tryll_classify_intent_params.md)



[More...](#detailed-description)

* `#include <TryllClassifyIntentParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  int32 | [**DiagnosticTopk**](#variable-diagnostictopk)   = `5`<br> |
|  [**FTryllStoragePath**](struct_f_tryll_storage_path.md) | [**EmbeddedStringStorage**](#variable-embeddedstringstorage)  <br> |
|  FString | [**Filter**](#variable-filter)  <br> |
|  FString | [**FoundExit**](#variable-foundexit)  <br> |
|  FString | [**MetadataField**](#variable-metadatafield)  <br> |
|  FString | [**NotFoundExit**](#variable-notfoundexit)  <br> |
|  float | [**Threshold**](#variable-threshold)   = `0.25f`<br> |
|  bool | [**bNotifyClient**](#variable-bnotifyclient)   = `false`<br> |
|  bool | [**bOverrideFilter**](#variable-boverridefilter)   = `false`<br> |
|  bool | [**bOverrideMetadataField**](#variable-boverridemetadatafield)   = `false`<br> |
































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


Intent classification node (embedding-based, top-1 nearest-neighbour). Searches a labelled embedded storage and attaches an IntentionComponent. 


    
## Public Attributes Documentation




### variable DiagnosticTopk 

```C++
int32 UTryllClassifyIntentParams::DiagnosticTopk;
```



Hits to fetch for diagnostic purposes (routing always uses top-1). 


        

<hr>



### variable EmbeddedStringStorage 

```C++
FTryllStoragePath UTryllClassifyIntentParams::EmbeddedStringStorage;
```



Named embedded string storage. Structural. 


        

<hr>



### variable Filter 

```C++
FString UTryllClassifyIntentParams::Filter;
```




<hr>



### variable FoundExit 

```C++
FString UTryllClassifyIntentParams::FoundExit;
```



Exit taken when classification produced a label above threshold. Empty string = END. Graph exit "found" — target node name; empty = END. 


        

<hr>



### variable MetadataField 

```C++
FString UTryllClassifyIntentParams::MetadataField;
```




<hr>



### variable NotFoundExit 

```C++
FString UTryllClassifyIntentParams::NotFoundExit;
```



Exit taken when no label cleared the threshold. Empty string = END. Graph exit "not\_found" — target node name; empty = END. 


        

<hr>



### variable Threshold 

```C++
float UTryllClassifyIntentParams::Threshold;
```



Maximum cosine distance; hits above this are treated as not\_found. 


        

<hr>



### variable bNotifyClient 

```C++
bool UTryllClassifyIntentParams::bNotifyClient;
```



When true, fire OnNodeEvent("intent\_classified", …) on the found path. 


        

<hr>



### variable bOverrideFilter 

```C++
bool UTryllClassifyIntentParams::bOverrideFilter;
```



JSON filter applied during search. Empty = no filter. Mutable. Set to true to override the inherited Filter value. 


        

<hr>



### variable bOverrideMetadataField 

```C++
bool UTryllClassifyIntentParams::bOverrideMetadataField;
```



Storage metadata field whose string value becomes the intent label. Set to true to override the inherited MetadataField value. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllClassifyIntentParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllClassifyIntentParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllClassifyIntentParams.h`

