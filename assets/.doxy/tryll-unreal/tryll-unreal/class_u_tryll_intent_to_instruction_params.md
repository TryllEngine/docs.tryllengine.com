

# Class UTryllIntentToInstructionParams



[**ClassList**](annotated.md) **>** [**UTryllIntentToInstructionParams**](class_u_tryll_intent_to_instruction_params.md)



[More...](#detailed-description)

* `#include <TryllIntentToInstructionParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**InstructionName**](#variable-instructionname)  <br> |
|  FString | [**StringStorage**](#variable-stringstorage)  <br> |
|  bool | [**bOverrideInlineKeys**](#variable-boverrideinlinekeys)   = `false`<br> |
|  bool | [**bOverrideInlineStrings**](#variable-boverrideinlinestrings)   = `false`<br> |
|  bool | [**bOverrideInstructionName**](#variable-boverrideinstructionname)   = `false`<br> |
|  bool | [**bOverrideStringStorage**](#variable-boverridestringstorage)   = `false`<br> |
































## Public Functions

| Type | Name |
| ---: | :--- |
| virtual ETryllNodeType | [**GetNodeType**](#function-getnodetype) () override const<br> |
| virtual flatbuffers::Offset&lt; void &gt; | [**Pack**](#function-pack) (flatbuffers::FlatBufferBuilder & Fbb, Tryll::NodeParams::NodeParams & OutType) override const<br> |
|   | [**UPROPERTY**](#function-uproperty-13) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|IntentToInstruction", meta=(EditCondition="bOverrideInlineKeys")) <br> |
|   | [**UPROPERTY**](#function-uproperty-23) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|IntentToInstruction", meta=(EditCondition="bOverrideInlineStrings")) <br> |
|   | [**UPROPERTY**](#function-uproperty-33) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|Exits", meta=(GetOptions="GetExitTargetOptions")) <br> |


## Public Functions inherited from UTryllNodeParamsBase

See [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)

| Type | Name |
| ---: | :--- |
| virtual TArray&lt; FString &gt; | [**GetExitTargetOptions**](class_u_tryll_node_params_base.md#function-getexittargetoptions) () const<br> |
| virtual ETryllNodeType | [**GetNodeType**](class_u_tryll_node_params_base.md#function-getnodetype) () const<br> |
| virtual flatbuffers::Offset&lt; void &gt; | [**Pack**](class_u_tryll_node_params_base.md#function-pack) (flatbuffers::FlatBufferBuilder & Fbb, ::Tryll::NodeParams::NodeParams & OutType) const<br> |
|  ETryllNodeType return | [**static\_cast**](class_u_tryll_node_params_base.md#function-static_cast) (0) <br> |






















































## Detailed Description


Maps an intent label to an instruction text from a Map-kind string storage. Resolution priority (factory): named string\_storage → inline\_keys + inline\_strings (Map-kind in-memory storage built from the equal-length pair). 


    
## Public Attributes Documentation




### variable InstructionName 

```C++
FString UTryllIntentToInstructionParams::InstructionName;
```




<hr>



### variable StringStorage 

```C++
FString UTryllIntentToInstructionParams::StringStorage;
```




<hr>



### variable bOverrideInlineKeys 

```C++
bool UTryllIntentToInstructionParams::bOverrideInlineKeys;
```



Inline map keys (equal-length with inline\_strings). Structural because they materialise the in-memory Map storage at construction. 


        

<hr>



### variable bOverrideInlineStrings 

```C++
bool UTryllIntentToInstructionParams::bOverrideInlineStrings;
```



Inline map values (equal-length with inline\_keys). Structural. 


        

<hr>



### variable bOverrideInstructionName 

```C++
bool UTryllIntentToInstructionParams::bOverrideInstructionName;
```



Name used as the Mustache variable suffix (instruction\_&lt;instruction\_name&gt;) on the attached InstructionComponent. Set to true to override the inherited InstructionName value. 


        

<hr>



### variable bOverrideStringStorage 

```C++
bool UTryllIntentToInstructionParams::bOverrideStringStorage;
```



Named session Map-kind string storage. Mutable — rebinds the storage. Set to true to override the inherited StringStorage value. 


        

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



### function UPROPERTY [1/3]

```C++
UTryllIntentToInstructionParams::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|IntentToInstruction",
    meta=(EditCondition="bOverrideInlineKeys")
) 
```




<hr>



### function UPROPERTY [2/3]

```C++
UTryllIntentToInstructionParams::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|IntentToInstruction",
    meta=(EditCondition="bOverrideInlineStrings")
) 
```




<hr>



### function UPROPERTY [3/3]

```C++
UTryllIntentToInstructionParams::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|Exits",
    meta=(GetOptions="GetExitTargetOptions")
) 
```



Default exit target after mapping an intent to an instruction. Empty string = END. Graph exit "default" — target node name; empty = END. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/Generated/Nodes/TryllIntentToInstructionParams.h`

