

# Class UTryllHumanMessageGuardrailParams



[**ClassList**](annotated.md) **>** [**UTryllHumanMessageGuardrailParams**](class_u_tryll_human_message_guardrail_params.md)



[More...](#detailed-description)

* `#include <TryllHumanMessageGuardrailParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  TArray&lt; FString &gt; | [**InlineStrings**](#variable-inlinestrings)  <br> |
|  FString | [**NotTriggeredExit**](#variable-nottriggeredexit)  <br> |
|  [**FTryllStoragePath**](struct_f_tryll_storage_path.md) | [**StringStorage**](#variable-stringstorage)  <br> |
|  FString | [**TriggeredExit**](#variable-triggeredexit)  <br> |
































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


Guardrail node backed by session string storage. Blocks turns whose human message matches a pattern in the storage. Resolution priority (factory): named string\_storage → inline\_strings → server-wide default (default-guardrail-patterns.txt from server-config). 


    
## Public Attributes Documentation




### variable InlineStrings 

```C++
TArray<FString> UTryllHumanMessageGuardrailParams::InlineStrings;
```



Inline pattern list. Structural because it materialises the underlying StringStorage at construction; rebinding goes through string\_storage. 


        

<hr>



### variable NotTriggeredExit 

```C++
FString UTryllHumanMessageGuardrailParams::NotTriggeredExit;
```



Exit taken when no pattern matches. Empty string = END. Graph exit "not\_triggered" — target node name; empty = END. 


        

<hr>



### variable StringStorage 

```C++
FTryllStoragePath UTryllHumanMessageGuardrailParams::StringStorage;
```



Named session string storage. Mutable — rebinds the underlying storage. 


        

<hr>



### variable TriggeredExit 

```C++
FString UTryllHumanMessageGuardrailParams::TriggeredExit;
```



Exit taken when the human message matches a guardrail pattern. Empty string = END. Graph exit "triggered" — target node name; empty = END. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllHumanMessageGuardrailParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllHumanMessageGuardrailParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllHumanMessageGuardrailParams.h`

