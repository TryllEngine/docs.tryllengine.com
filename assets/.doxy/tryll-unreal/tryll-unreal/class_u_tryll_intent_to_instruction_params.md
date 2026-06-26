

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
|  FString | [**InstructionName**](#variable-instructionname)  <br> |
|  [**FTryllStoragePath**](struct_f_tryll_storage_path.md) | [**StringStorage**](#variable-stringstorage)  <br> |
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



### variable InstructionName 

```C++
FString UTryllIntentToInstructionParams::InstructionName;
```




<hr>



### variable StringStorage 

```C++
FTryllStoragePath UTryllIntentToInstructionParams::StringStorage;
```



Named session Map-kind string storage. Mutable — rebinds the storage. 


        

<hr>



### variable bOverrideInstructionName 

```C++
bool UTryllIntentToInstructionParams::bOverrideInstructionName;
```



Name used as the Mustache variable suffix (instruction\_&lt;instruction\_name&gt;) on the attached InstructionComponent. Set to true to override the inherited InstructionName value. 


        

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
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllIntentToInstructionParams.h`

