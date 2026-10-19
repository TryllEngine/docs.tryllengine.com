

# Class UTryllTransformParams



[**ClassList**](annotated.md) **>** [**UTryllTransformParams**](class_u_tryll_transform_params.md)



[More...](#detailed-description)

* `#include <TryllTransformParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**DefaultExit**](#variable-defaultexit)  <br> |
|  [**FTryllSlotName**](struct_f_tryll_slot_name.md) | [**OutputName**](#variable-outputname)  <br> |
|  FString | [**Template**](#variable-template)  <br> |
|  FString | [**TemplateTextId**](#variable-templatetextid)  <br> |
|  bool | [**bOverrideOutputName**](#variable-boverrideoutputname)   = `false`<br> |
|  bool | [**bTemplateLocalized**](#variable-btemplatelocalized)   = `false`<br> |
































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
| virtual bool | [**IsTtsConsumer**](class_u_tryll_node_params_base.md#function-isttsconsumer) () const<br> |
| virtual flatbuffers::Offset&lt; void &gt; | [**Pack**](class_u_tryll_node_params_base.md#function-pack) (flatbuffers::FlatBufferBuilder & Fbb, ::Tryll::NodeParams::NodeParams & OutType) const<br> |
| virtual void | [**PostLoad**](class_u_tryll_node_params_base.md#function-postload) () override<br> |
| virtual void | [**Serialize**](class_u_tryll_node_params_base.md#function-serialize) (FArchive & Ar) override<br> |
|  ETryllNodeType return | [**static\_cast**](class_u_tryll_node_params_base.md#function-static_cast) (0) <br> |




## Public Static Functions inherited from UTryllNodeParamsBase

See [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)

| Type | Name |
| ---: | :--- |
|  const [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) \* | [**FindEnclosingGraph**](class_u_tryll_node_params_base.md#function-findenclosinggraph) (const UObject \* Obj) <br> |












































## Protected Functions inherited from UTryllNodeParamsBase

See [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)

| Type | Name |
| ---: | :--- |
| virtual void | [**ApplyOldDefaults**](class_u_tryll_node_params_base.md#function-applyolddefaults) (int32 SavedVersion) <br> |
| virtual bool | [**UpgradeFrom**](class_u_tryll_node_params_base.md#function-upgradefrom) (int32 SavedVersion) <br> |






## Detailed Description


Model-free input-composition node. Renders a Mustache template over `user_message` and the current turn's slots into its own slot; performs no inference and never emits on the wire (`send = None`, `history_role = none`, fixed). Typical use: query rewriting, fan-out composition feeding a downstream Retrieve/Generate `input`. 


    
## Public Attributes Documentation




### variable DefaultExit 

```C++
FString UTryllTransformParams::DefaultExit;
```



Default exit target after rendering. Empty string = END. Graph exit "default" — target node name; empty = END. 


        

<hr>



### variable OutputName 

```C++
FTryllSlotName UTryllTransformParams::OutputName;
```



Effective output slot name for this node. Empty = the node's own name. 


        

<hr>



### variable Template 

```C++
FString UTryllTransformParams::Template;
```




<hr>



### variable TemplateTextId 

```C++
FString UTryllTransformParams::TemplateTextId;
```



Text ID in the workflow text table. The provider's text is what is sent. 


        

<hr>



### variable bOverrideOutputName 

```C++
bool UTryllTransformParams::bOverrideOutputName;
```



Name this node's output slot is stored under. Empty = the node's own name. Structural: immutable after creation — renaming would re-wire the slot dataflow that is validated once at agent creation. Set to true to override the inherited OutputName value. 


        

<hr>



### variable bTemplateLocalized 

```C++
bool UTryllTransformParams::bTemplateLocalized;
```



Mustache template rendered at execution time against the user\_message tag, slot.&lt;name&gt; tags, knowledge\_&lt;source&gt; tags, and the #instructions iterable block. Ticked: TemplateTextId is resolved by the text provider and that text is sent. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllTransformParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllTransformParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllTransformParams.h`

