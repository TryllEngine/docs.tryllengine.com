

# Class UTryllIntentToInstructionParams



[**ClassList**](annotated.md) **>** [**UTryllIntentToInstructionParams**](class_u_tryll_intent_to_instruction_params.md)



[More...](#detailed-description)

* `#include <TryllIntentToInstructionParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**DefaultExit**](#variable-defaultexit)  <br> |
|  TArray&lt; FString &gt; | [**InlineKeys**](#variable-inlinekeys)  <br> |
|  TArray&lt; FString &gt; | [**InlineStrings**](#variable-inlinestrings)  <br> |
|  [**FTryllSlotName**](struct_f_tryll_slot_name.md) | [**OutputName**](#variable-outputname)  <br> |
|  [**FTryllStoragePath**](struct_f_tryll_storage_path.md) | [**StringStorage**](#variable-stringstorage)  <br> |
|  bool | [**bOverrideOutputName**](#variable-boverrideoutputname)   = `false`<br> |
































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


Maps an intent label to an instruction text from a Map-kind string storage. Resolution priority (factory): named string\_storage → inline\_keys + inline\_strings (Map-kind in-memory storage built from the equal-length pair). 


    
## Public Attributes Documentation




### variable DefaultExit 

```C++
FString UTryllIntentToInstructionParams::DefaultExit;
```



Default exit target after mapping an intent to an instruction. Empty string = END. Graph exit "default" — target node name; empty = END. 


        

<hr>



### variable InlineKeys 

```C++
TArray<FString> UTryllIntentToInstructionParams::InlineKeys;
```



Inline map keys (equal-length with inline\_strings). Structural because they materialise the in-memory Map storage at construction. 


        

<hr>



### variable InlineStrings 

```C++
TArray<FString> UTryllIntentToInstructionParams::InlineStrings;
```



Inline map values (equal-length with inline\_keys). Structural. 


        

<hr>



### variable OutputName 

```C++
FTryllSlotName UTryllIntentToInstructionParams::OutputName;
```



Effective output slot name for this node. Empty = the node's own name. 


        

<hr>



### variable StringStorage 

```C++
FTryllStoragePath UTryllIntentToInstructionParams::StringStorage;
```



Named session Map-kind string storage. Mutable — rebinds the storage. 


        

<hr>



### variable bOverrideOutputName 

```C++
bool UTryllIntentToInstructionParams::bOverrideOutputName;
```



Name this node's output slot is stored under. Empty = the node's own name. Downstream templates look it up via slot.&lt;name&gt;. Structural: immutable after creation — renaming would re-wire the slot dataflow that is validated once at agent creation. Set to true to override the inherited OutputName value. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllIntentToInstructionParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllIntentToInstructionParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllIntentToInstructionParams.h`

