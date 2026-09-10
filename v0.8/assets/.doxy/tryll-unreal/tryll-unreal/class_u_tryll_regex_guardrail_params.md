

# Class UTryllRegexGuardrailParams



[**ClassList**](annotated.md) **>** [**UTryllRegexGuardrailParams**](class_u_tryll_regex_guardrail_params.md)



[More...](#detailed-description)

* `#include <TryllRegexGuardrailParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  TArray&lt; FString &gt; | [**InlineStrings**](#variable-inlinestrings)  <br> |
|  [**FTryllSlotInput**](struct_f_tryll_slot_input.md) | [**Input**](#variable-input)  <br> |
|  FString | [**NotTriggeredExit**](#variable-nottriggeredexit)  <br> |
|  [**FTryllStoragePath**](struct_f_tryll_storage_path.md) | [**StringStorage**](#variable-stringstorage)  <br> |
|  FString | [**TriggeredExit**](#variable-triggeredexit)  <br> |
|  bool | [**bOverrideInput**](#variable-boverrideinput)   = `false`<br> |
































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


Regex guardrail node backed by session string storage. Branches on whether the node's input slot text matches a pattern in the storage. Resolution priority (factory): named string\_storage → inline\_strings → server-wide default (default-guardrail-patterns.txt from server-config). 


    
## Public Attributes Documentation




### variable InlineStrings 

```C++
TArray<FString> UTryllRegexGuardrailParams::InlineStrings;
```



Inline pattern list. Structural because it materialises the underlying StringStorage at construction; rebinding goes through string\_storage. 


        

<hr>



### variable Input 

```C++
FTryllSlotInput UTryllRegexGuardrailParams::Input;
```



Upstream inter-node slot this field consumes. Empty = context default. 


        

<hr>



### variable NotTriggeredExit 

```C++
FString UTryllRegexGuardrailParams::NotTriggeredExit;
```



Exit taken when no pattern matches. Empty string = END. Graph exit "not\_triggered" — target node name; empty = END. 


        

<hr>



### variable StringStorage 

```C++
FTryllStoragePath UTryllRegexGuardrailParams::StringStorage;
```



Named session string storage. Mutable — rebinds the underlying storage. 


        

<hr>



### variable TriggeredExit 

```C++
FString UTryllRegexGuardrailParams::TriggeredExit;
```



Exit taken when the input slot text matches a guardrail pattern. Empty string = END. Graph exit "triggered" — target node name; empty = END. 


        

<hr>



### variable bOverrideInput 

```C++
bool UTryllRegexGuardrailParams::bOverrideInput;
```



Slot name this node matches against. Empty = "user\_message" (same default as every other input-bearing node). Structural: immutable after creation — rebinding would re-wire the slot dataflow that is validated once at agent creation. Set to true to override the inherited Input value. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllRegexGuardrailParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllRegexGuardrailParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllRegexGuardrailParams.h`

