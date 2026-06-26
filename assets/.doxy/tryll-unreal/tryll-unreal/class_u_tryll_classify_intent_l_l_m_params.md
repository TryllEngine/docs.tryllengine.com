

# Class UTryllClassifyIntentLLMParams



[**ClassList**](annotated.md) **>** [**UTryllClassifyIntentLLMParams**](class_u_tryll_classify_intent_l_l_m_params.md)



[More...](#detailed-description)

* `#include <TryllClassifyIntentLLMParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**FoundExit**](#variable-foundexit)  <br> |
|  int32 | [**HistoryTurns**](#variable-historyturns)   = `2`<br> |
|  FString | [**IntentsIds**](#variable-intentsids)  <br> |
|  FString | [**IntentsPrompt**](#variable-intentsprompt)  <br> |
|  float | [**Margin**](#variable-margin)   = `0.15f`<br> |
|  [**FTryllModelName**](struct_f_tryll_model_name.md) | [**ModelName**](#variable-modelname)  <br> |
|  FString | [**NotFoundExit**](#variable-notfoundexit)  <br> |
|  FString | [**NotFoundIntent**](#variable-notfoundintent)  <br> |
|  FString | [**SystemPrompt**](#variable-systemprompt)  <br> |
|  float | [**Threshold**](#variable-threshold)   = `0.5f`<br> |
|  bool | [**bNotifyClient**](#variable-bnotifyclient)   = `false`<br> |
|  bool | [**bOverrideIntentsIds**](#variable-boverrideintentsids)   = `false`<br> |
|  bool | [**bOverrideIntentsPrompt**](#variable-boverrideintentsprompt)   = `false`<br> |
|  bool | [**bOverrideModelName**](#variable-boverridemodelname)   = `false`<br> |
|  bool | [**bOverrideNotFoundIntent**](#variable-boverridenotfoundintent)   = `false`<br> |
|  bool | [**bOverrideSystemPrompt**](#variable-boverridesystemprompt)   = `false`<br> |
































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


Intent classification node (LLM first-token logprob over letter candidates). Scores labels via a dedicated LLM and attaches an IntentionComponent. 


    
## Public Attributes Documentation




### variable FoundExit 

```C++
FString UTryllClassifyIntentLLMParams::FoundExit;
```



Exit taken when classification produced a label above threshold/margin. Empty string = END. Graph exit "found" — target node name; empty = END. 


        

<hr>



### variable HistoryTurns 

```C++
int32 UTryllClassifyIntentLLMParams::HistoryTurns;
```



Number of dialog history turns included in the classification context. 


        

<hr>



### variable IntentsIds 

```C++
FString UTryllClassifyIntentLLMParams::IntentsIds;
```




<hr>



### variable IntentsPrompt 

```C++
FString UTryllClassifyIntentLLMParams::IntentsPrompt;
```




<hr>



### variable Margin 

```C++
float UTryllClassifyIntentLLMParams::Margin;
```



Minimum margin between top and second label probabilities. 


        

<hr>



### variable ModelName 

```C++
FTryllModelName UTryllClassifyIntentLLMParams::ModelName;
```




<hr>



### variable NotFoundExit 

```C++
FString UTryllClassifyIntentLLMParams::NotFoundExit;
```



Exit taken when no label cleared the threshold/margin. Empty string = END. Graph exit "not\_found" — target node name; empty = END. 


        

<hr>



### variable NotFoundIntent 

```C++
FString UTryllClassifyIntentLLMParams::NotFoundIntent;
```




<hr>



### variable SystemPrompt 

```C++
FString UTryllClassifyIntentLLMParams::SystemPrompt;
```




<hr>



### variable Threshold 

```C++
float UTryllClassifyIntentLLMParams::Threshold;
```



First-token probability threshold for the winning label. 


        

<hr>



### variable bNotifyClient 

```C++
bool UTryllClassifyIntentLLMParams::bNotifyClient;
```



When true, fire OnNodeEvent("intent\_classified", …) on the found path. 


        

<hr>



### variable bOverrideIntentsIds 

```C++
bool UTryllClassifyIntentLLMParams::bOverrideIntentsIds;
```



Comma-separated intent IDs (e.g. "greet,farewell,other"). Structural because label-to-token-ID mapping is built at construction. Set to true to override the inherited IntentsIds value. 


        

<hr>



### variable bOverrideIntentsPrompt 

```C++
bool UTryllClassifyIntentLLMParams::bOverrideIntentsPrompt;
```



Comma-separated intent prompt labels / descriptions. Structural because the projection scaffolding and label-token mapping are built at construction. Set to true to override the inherited IntentsPrompt value. 


        

<hr>



### variable bOverrideModelName 

```C++
bool UTryllClassifyIntentLLMParams::bOverrideModelName;
```



Model catalog name. Structural because the token-ID mapping is built from the model's vocabulary at construction. Set to true to override the inherited ModelName value. 


        

<hr>



### variable bOverrideNotFoundIntent 

```C++
bool UTryllClassifyIntentLLMParams::bOverrideNotFoundIntent;
```



Intent ID returned when no label clears the threshold/margin. Set to true to override the inherited NotFoundIntent value. 


        

<hr>



### variable bOverrideSystemPrompt 

```C++
bool UTryllClassifyIntentLLMParams::bOverrideSystemPrompt;
```



Prepended before the user turn during classification projection. Set to true to override the inherited SystemPrompt value. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllClassifyIntentLLMParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllClassifyIntentLLMParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllClassifyIntentLLMParams.h`

