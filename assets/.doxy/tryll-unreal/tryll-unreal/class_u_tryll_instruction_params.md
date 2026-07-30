

# Class UTryllInstructionParams



[**ClassList**](annotated.md) **>** [**UTryllInstructionParams**](class_u_tryll_instruction_params.md)



[More...](#detailed-description)

* `#include <TryllInstructionParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**DefaultExit**](#variable-defaultexit)  <br> |
|  FString | [**Instruction**](#variable-instruction)  <br> |
|  [**FTryllSlotName**](struct_f_tryll_slot_name.md) | [**OutputName**](#variable-outputname)  <br> |
|  bool | [**bOverrideInstruction**](#variable-boverrideinstruction)   = `false`<br> |
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


Plain instruction node. Attaches a kind=Instruction SlotComponent to the current interaction so downstream Mustache templates can reference it. 


    
## Public Attributes Documentation




### variable DefaultExit 

```C++
FString UTryllInstructionParams::DefaultExit;
```



Default exit target after attaching the instruction. Empty string = END. Graph exit "default" — target node name; empty = END. 


        

<hr>



### variable Instruction 

```C++
FString UTryllInstructionParams::Instruction;
```




<hr>



### variable OutputName 

```C++
FTryllSlotName UTryllInstructionParams::OutputName;
```



Effective output slot name for this node. Empty = the node's own name. 


        

<hr>



### variable bOverrideInstruction 

```C++
bool UTryllInstructionParams::bOverrideInstruction;
```



The instruction text. Mutable. Set to true to override the inherited Instruction value. 


        

<hr>



### variable bOverrideOutputName 

```C++
bool UTryllInstructionParams::bOverrideOutputName;
```



Name this node's output slot is stored under. Empty = the node's own name. Downstream templates look it up via slot.&lt;name&gt;. Structural: immutable after creation — renaming would re-wire the slot dataflow that is validated once at agent creation. Set to true to override the inherited OutputName value. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllInstructionParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllInstructionParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllInstructionParams.h`

