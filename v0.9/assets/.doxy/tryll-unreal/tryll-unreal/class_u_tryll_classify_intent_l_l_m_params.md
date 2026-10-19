

# Class UTryllClassifyIntentLLMParams



[**ClassList**](annotated.md) **>** [**UTryllClassifyIntentLLMParams**](class_u_tryll_classify_intent_l_l_m_params.md)



[More...](#detailed-description)

* `#include <TryllClassifyIntentLLMParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  [**FTryllModelAdaptation**](struct_f_tryll_model_adaptation.md) | [**Adaptation**](#variable-adaptation)  <br> |
|  [**FTryllContextOverrides**](struct_f_tryll_context_overrides.md) | [**Context**](#variable-context)  <br> |
|  int32 | [**ContextSize**](#variable-contextsize)   = `0`<br> |
|  FString | [**FoundExit**](#variable-foundexit)  <br> |
|  int32 | [**HistoryTurns**](#variable-historyturns)   = `2`<br> |
|  [**FTryllSlotInput**](struct_f_tryll_slot_input.md) | [**Input**](#variable-input)  <br> |
|  TArray&lt; [**FTryllClassifyIntentEntry**](struct_f_tryll_classify_intent_entry.md) &gt; | [**Intents**](#variable-intents)  <br> |
|  FString | [**IntentsIds\_DEPRECATED**](#variable-intentsids_deprecated)  <br> |
|  FString | [**IntentsPrompt\_DEPRECATED**](#variable-intentsprompt_deprecated)  <br> |
|  float | [**Margin**](#variable-margin)   = `0.15f`<br> |
|  [**FTryllModelName**](struct_f_tryll_model_name.md) | [**ModelName**](#variable-modelname)  <br> |
|  FString | [**NotFoundExit**](#variable-notfoundexit)  <br> |
|  FString | [**NotFoundIntent**](#variable-notfoundintent)  <br> |
|  ETryllNotifyClient | [**NotifyClient**](#variable-notifyclient)   = `ETryllNotifyClient::Disabled`<br> |
|  [**FTryllSlotName**](struct_f_tryll_slot_name.md) | [**OutputName**](#variable-outputname)  <br> |
|  FString | [**SystemPrompt**](#variable-systemprompt)  <br> |
|  FString | [**SystemPromptTextId**](#variable-systemprompttextid)  <br> |
|  float | [**Temperature**](#variable-temperature)   = `0.0f`<br> |
|  float | [**Threshold**](#variable-threshold)   = `0.5f`<br> |
|  bool | [**bOverrideInput**](#variable-boverrideinput)   = `false`<br> |
|  bool | [**bOverrideIntentsIds\_DEPRECATED**](#variable-boverrideintentsids_deprecated)   = `false`<br> |
|  bool | [**bOverrideIntentsPrompt\_DEPRECATED**](#variable-boverrideintentsprompt_deprecated)   = `false`<br> |
|  bool | [**bOverrideModelName**](#variable-boverridemodelname)   = `false`<br> |
|  bool | [**bOverrideOutputName**](#variable-boverrideoutputname)   = `false`<br> |
|  bool | [**bSystemPromptLocalized**](#variable-bsystempromptlocalized)   = `false`<br> |
































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


Intent classification node (LLM first-token logprob over letter candidates). Scores labels via a dedicated LLM, attaches an IntentionComponent on found, and writes the decision to slot.&lt;output\_name&gt;. 


    
## Public Attributes Documentation




### variable Adaptation 

```C++
FTryllModelAdaptation UTryllClassifyIntentLLMParams::Adaptation;
```



Changes how the model behaves for this node (LoRA adapters; later control vectors). Null = base model. Structural: applied to the llama context at creation, so a change means a new context. Appended last so existing field IDs do not move. 


        

<hr>



### variable Context 

```C++
FTryllContextOverrides UTryllClassifyIntentLLMParams::Context;
```



Backend context construction. Null table inherits, except CreateContext still defaults `n_outputs_max` to 1. Structural. 


        

<hr>



### variable ContextSize 

```C++
int32 UTryllClassifyIntentLLMParams::ContextSize;
```



KV-cache / context window (n\_ctx) for this node in tokens. 0 = node default 2048, else the model variant's context\_size, else server default\_n\_ctx. Validated against the model's trained maximum at agent creation. 


        

<hr>



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



### variable Input 

```C++
FTryllSlotInput UTryllClassifyIntentLLMParams::Input;
```



Upstream inter-node slot this field consumes. Empty = context default. 


        

<hr>



### variable Intents 

```C++
TArray<FTryllClassifyIntentEntry> UTryllClassifyIntentLLMParams::Intents;
```



Intent list: id + prompt pairs. Entry 0 is letter A. 2-26 unique ids, non-empty prompts. Structural because the projection scaffolding and label-token mapping are built at construction. 


        

<hr>



### variable IntentsIds\_DEPRECATED 

```C++
FString UTryllClassifyIntentLLMParams::IntentsIds_DEPRECATED;
```




<hr>



### variable IntentsPrompt\_DEPRECATED 

```C++
FString UTryllClassifyIntentLLMParams::IntentsPrompt_DEPRECATED;
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



Sink intent. When it wins, the node exits not\_found. Written to the output slot on every not\_found path. One of the intent ids, the reserved "#uncertain", or empty. 


        

<hr>



### variable NotifyClient 

```C++
ETryllNotifyClient UTryllClassifyIntentLLMParams::NotifyClient;
```



When to fire OnNodeEvent("intent\_llm\_classified", …). 


        

<hr>



### variable OutputName 

```C++
FTryllSlotName UTryllClassifyIntentLLMParams::OutputName;
```



Effective output slot name for this node. Empty = the node's own name. 


        

<hr>



### variable SystemPrompt 

```C++
FString UTryllClassifyIntentLLMParams::SystemPrompt;
```




<hr>



### variable SystemPromptTextId 

```C++
FString UTryllClassifyIntentLLMParams::SystemPromptTextId;
```



Text ID in the workflow text table. The provider's text is what is sent. 


        

<hr>



### variable Temperature 

```C++
float UTryllClassifyIntentLLMParams::Temperature;
```



Softmax temperature applied to scored letter probabilities before threshold/margin. 0 = 1.0 (identity). Mutable. 


        

<hr>



### variable Threshold 

```C++
float UTryllClassifyIntentLLMParams::Threshold;
```



First-token probability threshold for the winning label. 


        

<hr>



### variable bOverrideInput 

```C++
bool UTryllClassifyIntentLLMParams::bOverrideInput;
```



Slot name this node consumes as its primary text for the current turn. Empty = "user\_message". Structural: immutable after creation — rebinding would re-wire the slot dataflow that is validated once at agent creation. Set to true to override the inherited Input value. 


        

<hr>



### variable bOverrideIntentsIds\_DEPRECATED 

```C++
bool UTryllClassifyIntentLLMParams::bOverrideIntentsIds_DEPRECATED;
```




<hr>



### variable bOverrideIntentsPrompt\_DEPRECATED 

```C++
bool UTryllClassifyIntentLLMParams::bOverrideIntentsPrompt_DEPRECATED;
```




<hr>



### variable bOverrideModelName 

```C++
bool UTryllClassifyIntentLLMParams::bOverrideModelName;
```



Model catalog name. Structural because the token-ID mapping is built from the model's vocabulary at construction. Set to true to override the inherited ModelName value. 


        

<hr>



### variable bOverrideOutputName 

```C++
bool UTryllClassifyIntentLLMParams::bOverrideOutputName;
```



Name of the slot this node writes on every path. Empty = node name. Holds the winning intent id on found, and not\_found\_intent (possibly empty) on not\_found. Structural: immutable after creation — renaming would re-wire the slot dataflow that is validated once at agent creation. Set to true to override the inherited OutputName value. 


        

<hr>



### variable bSystemPromptLocalized 

```C++
bool UTryllClassifyIntentLLMParams::bSystemPromptLocalized;
```



Mustache template for the classifier's system message. Required: empty or unparsable templates fail agent creation and ChangeAgentParam. Must reference intents\_block (the letter-prefixed list built from the resolved prompts). letters (e.g. "A, B, C") is optional — padded and triple-brace forms count. Ticked: SystemPromptTextId is resolved by the text provider and that text is sent. 


        

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
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllClassifyIntentLLMParams.h`

