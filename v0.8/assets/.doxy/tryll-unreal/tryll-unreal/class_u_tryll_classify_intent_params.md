

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
|  [**FTryllSlotInput**](struct_f_tryll_slot_input.md) | [**Input**](#variable-input)  <br> |
|  FString | [**MetadataField**](#variable-metadatafield)  <br> |
|  FString | [**NotFoundExit**](#variable-notfoundexit)  <br> |
|  ETryllNotifyClient | [**NotifyClient**](#variable-notifyclient)   = `ETryllNotifyClient::Disabled`<br> |
|  float | [**Threshold**](#variable-threshold)   = `0.25f`<br> |
|  bool | [**bOverrideFilter**](#variable-boverridefilter)   = `false`<br> |
|  bool | [**bOverrideInput**](#variable-boverrideinput)   = `false`<br> |
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
|  FString | [**GetOwnerNodeName**](class_u_tryll_node_params_base.md#function-getownernodename) () const<br> |
| virtual flatbuffers::Offset&lt; void &gt; | [**Pack**](class_u_tryll_node_params_base.md#function-pack) (flatbuffers::FlatBufferBuilder & Fbb, ::Tryll::NodeParams::NodeParams & OutType) const<br> |
|  ETryllNodeType return | [**static\_cast**](class_u_tryll_node_params_base.md#function-static_cast) (0) <br> |




## Public Static Functions inherited from UTryllNodeParamsBase

See [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)

| Type | Name |
| ---: | :--- |
|  const [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) \* | [**FindEnclosingGraph**](class_u_tryll_node_params_base.md#function-findenclosinggraph) (const UObject \* Obj) <br> |


















































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



Exit taken when the top-1 hit's distance is at or below threshold. Empty string = END. Graph exit "found" — target node name; empty = END. 


        

<hr>



### variable Input 

```C++
FTryllSlotInput UTryllClassifyIntentParams::Input;
```



Upstream inter-node slot this field consumes. Empty = context default. 


        

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



### variable NotifyClient 

```C++
ETryllNotifyClient UTryllClassifyIntentParams::NotifyClient;
```



When to fire OnNodeEvent("intent\_classified", …). 


        

<hr>



### variable Threshold 

```C++
float UTryllClassifyIntentParams::Threshold;
```



Maximum cosine distance; hits above this are treated as not\_found. Cosine distance on normalized vectors is in [0, 2]. 


        

<hr>



### variable bOverrideFilter 

```C++
bool UTryllClassifyIntentParams::bOverrideFilter;
```



JSON filter applied during search. Empty = no filter. Mutable. Set to true to override the inherited Filter value. 


        

<hr>



### variable bOverrideInput 

```C++
bool UTryllClassifyIntentParams::bOverrideInput;
```



Slot name this node embeds and searches with. Empty = "user\_message". Structural: immutable after creation — rebinding would re-wire the slot dataflow that is validated once at agent creation. Set to true to override the inherited Input value. 


        

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
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllClassifyIntentParams.h`

