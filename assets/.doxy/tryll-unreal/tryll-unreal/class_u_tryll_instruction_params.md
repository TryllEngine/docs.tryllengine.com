

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
|  FString | [**InstructionName**](#variable-instructionname)  <br> |
|  bool | [**bOverrideInstruction**](#variable-boverrideinstruction)   = `false`<br> |
|  bool | [**bOverrideInstructionName**](#variable-boverrideinstructionname)   = `false`<br> |
































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


Plain instruction node. Attaches an InstructionComponent to the current interaction so downstream Mustache templates can reference it. 


    
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



### variable InstructionName 

```C++
FString UTryllInstructionParams::InstructionName;
```




<hr>



### variable bOverrideInstruction 

```C++
bool UTryllInstructionParams::bOverrideInstruction;
```



The instruction text. Mutable. Set to true to override the inherited Instruction value. 


        

<hr>



### variable bOverrideInstructionName 

```C++
bool UTryllInstructionParams::bOverrideInstructionName;
```



Name used as the Mustache variable suffix (instruction\_&lt;name&gt;). Set to true to override the inherited InstructionName value. 


        

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
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllInstructionParams.h`

