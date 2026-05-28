

# Class UTryllGenerateParams



[**ClassList**](annotated.md) **>** [**UTryllGenerateParams**](class_u_tryll_generate_params.md)



[More...](#detailed-description)

* `#include <TryllGenerateParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**ModelName**](#variable-modelname)  <br> |
|  ETryllPlacement | [**Placement**](#variable-placement)   = `ETryllPlacement::InPlaceOfUser`<br> |
|  [**FTryllSamplingOverrides**](struct_f_tryll_sampling_overrides.md) | [**Sampling**](#variable-sampling)  <br> |
|  FString | [**SystemPrompt**](#variable-systemprompt)  <br> |
|  FString | [**Template**](#variable-template)  <br> |
|  bool | [**bOverrideModelName**](#variable-boverridemodelname)   = `false`<br> |
|  bool | [**bOverrideSystemPrompt**](#variable-boverridesystemprompt)   = `false`<br> |
|  bool | [**bOverrideTemplate**](#variable-boverridetemplate)   = `false`<br> |
|  bool | [**bStream**](#variable-bstream)   = `false`<br> |
































## Public Functions

| Type | Name |
| ---: | :--- |
| virtual ETryllNodeType | [**GetNodeType**](#function-getnodetype) () override const<br> |
| virtual flatbuffers::Offset&lt; void &gt; | [**Pack**](#function-pack) (flatbuffers::FlatBufferBuilder & Fbb, Tryll::NodeParams::NodeParams & OutType) override const<br> |
|   | [**UPROPERTY**](#function-uproperty) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|Exits", meta=(GetOptions="GetExitTargetOptions")) <br> |


## Public Functions inherited from UTryllNodeParamsBase

See [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)

| Type | Name |
| ---: | :--- |
| virtual TArray&lt; FString &gt; | [**GetExitTargetOptions**](class_u_tryll_node_params_base.md#function-getexittargetoptions) () const<br> |
| virtual ETryllNodeType | [**GetNodeType**](class_u_tryll_node_params_base.md#function-getnodetype) () const<br> |
| virtual flatbuffers::Offset&lt; void &gt; | [**Pack**](class_u_tryll_node_params_base.md#function-pack) (flatbuffers::FlatBufferBuilder & Fbb, ::Tryll::NodeParams::NodeParams & OutType) const<br> |
|  ETryllNodeType return | [**static\_cast**](class_u_tryll_node_params_base.md#function-static_cast) (0) <br> |






















































## Detailed Description


LLM-driven text generation node. Renders a Mustache template, runs sampling, produces a streamed AnswerText. 


    
## Public Attributes Documentation




### variable ModelName 

```C++
FString UTryllGenerateParams::ModelName;
```




<hr>



### variable Placement 

```C++
ETryllPlacement UTryllGenerateParams::Placement;
```



Where the rendered template body is placed in the projected message stream. 


        

<hr>



### variable Sampling 

```C++
FTryllSamplingOverrides UTryllGenerateParams::Sampling;
```



Sparse sampling overrides applied on top of the model-catalog defaults. Sub-table change fires OnSamplingChanged, which re-resolves overrides against the cached model-default sampling. 


        

<hr>



### variable SystemPrompt 

```C++
FString UTryllGenerateParams::SystemPrompt;
```




<hr>



### variable Template 

```C++
FString UTryllGenerateParams::Template;
```




<hr>



### variable bOverrideModelName 

```C++
bool UTryllGenerateParams::bOverrideModelName;
```



Model catalog name. Empty = use the agent's default\_model\_name. Set to true to override the inherited ModelName value. 


        

<hr>



### variable bOverrideSystemPrompt 

```C++
bool UTryllGenerateParams::bOverrideSystemPrompt;
```



Prepended before the user turn during projection. Set to true to override the inherited SystemPrompt value. 


        

<hr>



### variable bOverrideTemplate 

```C++
bool UTryllGenerateParams::bOverrideTemplate;
```



Mustache template applied to the user-area message at projection time. Set to true to override the inherited Template value. 


        

<hr>



### variable bStream 

```C++
bool UTryllGenerateParams::bStream;
```



When true, emit answer text as streaming deltas; otherwise one final chunk. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllGenerateParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllGenerateParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>



### function UPROPERTY 

```C++
UTryllGenerateParams::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|Exits",
    meta=(GetOptions="GetExitTargetOptions")
) 
```



Default exit target — routes here after generation completes. Empty string = END (turn finishes). Graph exit "default" — target node name; empty = END. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/Generated/Nodes/TryllGenerateParams.h`

