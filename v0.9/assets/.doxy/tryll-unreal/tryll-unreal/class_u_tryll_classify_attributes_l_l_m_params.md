

# Class UTryllClassifyAttributesLLMParams



[**ClassList**](annotated.md) **>** [**UTryllClassifyAttributesLLMParams**](class_u_tryll_classify_attributes_l_l_m_params.md)



[More...](#detailed-description)

* `#include <TryllClassifyAttributesLLMParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  [**FTryllModelAdaptation**](struct_f_tryll_model_adaptation.md) | [**Adaptation**](#variable-adaptation)  <br> |
|  TArray&lt; [**FTryllClassifyAttributeDef**](struct_f_tryll_classify_attribute_def.md) &gt; | [**Attributes**](#variable-attributes)  <br> |
|  [**FTryllContextOverrides**](struct_f_tryll_context_overrides.md) | [**Context**](#variable-context)  <br> |
|  int32 | [**ContextSize**](#variable-contextsize)   = `0`<br> |
|  FString | [**DefaultExit**](#variable-defaultexit)  <br> |
|  int32 | [**HistoryTurns**](#variable-historyturns)   = `0`<br> |
|  [**FTryllSlotInput**](struct_f_tryll_slot_input.md) | [**Input**](#variable-input)  <br> |
|  [**FTryllModelName**](struct_f_tryll_model_name.md) | [**ModelName**](#variable-modelname)  <br> |
|  ETryllNotifyClient | [**NotifyClient**](#variable-notifyclient)   = `ETryllNotifyClient::Disabled`<br> |
|  ETryllAttributeFailureRoute | [**OnFailure**](#variable-onfailure)   = `ETryllAttributeFailureRoute::Default`<br> |
|  [**FTryllSlotName**](struct_f_tryll_slot_name.md) | [**OutputName**](#variable-outputname)  <br> |
|  float | [**Temperature**](#variable-temperature)   = `0.0f`<br> |
|  FString | [**TriggeredExit**](#variable-triggeredexit)  <br> |
|  bool | [**bOverrideInput**](#variable-boverrideinput)   = `false`<br> |
|  bool | [**bOverrideModelName**](#variable-boverridemodelname)   = `false`<br> |
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


Several specialist classifiers scored in one cached batched call. Each attribute writes slot.&lt;output\_name&gt;\_&lt;name&gt;. triggered\_exit is taken when any attribute's own rule fires. 


    
## Public Attributes Documentation




### variable Adaptation 

```C++
FTryllModelAdaptation UTryllClassifyAttributesLLMParams::Adaptation;
```



Changes how the model behaves for this node (LoRA adapters; later control vectors). Null = base model. Structural: applied to the llama context at creation, so a change means a new context. Appended last so existing field IDs do not move. 


        

<hr>



### variable Attributes 

```C++
TArray<FTryllClassifyAttributeDef> UTryllClassifyAttributesLLMParams::Attributes;
```



The attributes to classify. 1-16 entries, unique names. Shape, name, kind and choice id/prompt are structural; the hand-written Apply allows statement, prompt, decision and trigger edits. 


        

<hr>



### variable Context 

```C++
FTryllContextOverrides UTryllClassifyAttributesLLMParams::Context;
```



Backend context construction. The node derives n\_seq\_max and n\_outputs\_max from the attribute count. Structural. 


        

<hr>



### variable ContextSize 

```C++
int32 UTryllClassifyAttributesLLMParams::ContextSize;
```



KV-cache window in tokens. 0 = size from the rendered specialist prompts plus a per-attribute tail budget, else the classify default. 


        

<hr>



### variable DefaultExit 

```C++
FString UTryllClassifyAttributesLLMParams::DefaultExit;
```



Exit taken when no attribute triggers. Empty string = END. Graph exit "default" — target node name; empty = END. 


        

<hr>



### variable HistoryTurns 

```C++
int32 UTryllClassifyAttributesLLMParams::HistoryTurns;
```



Dialog turns included before the current user text. Default 0 until history quality is measured. 


        

<hr>



### variable Input 

```C++
FTryllSlotInput UTryllClassifyAttributesLLMParams::Input;
```



Upstream inter-node slot this field consumes. Empty = context default. 


        

<hr>



### variable ModelName 

```C++
FTryllModelName UTryllClassifyAttributesLLMParams::ModelName;
```




<hr>



### variable NotifyClient 

```C++
ETryllNotifyClient UTryllClassifyAttributesLLMParams::NotifyClient;
```



When to fire OnNodeEvent("attributes\_classified", …). OnFound means the triggered exit was taken. 


        

<hr>



### variable OnFailure 

```C++
ETryllAttributeFailureRoute UTryllClassifyAttributesLLMParams::OnFailure;
```



Exit taken when scoring fails. 


        

<hr>



### variable OutputName 

```C++
FTryllSlotName UTryllClassifyAttributesLLMParams::OutputName;
```



Effective output slot name for this node. Empty = the node's own name. 


        

<hr>



### variable Temperature 

```C++
float UTryllClassifyAttributesLLMParams::Temperature;
```



Softmax temperature for every attribute. 0 = 1.0 (identity). 


        

<hr>



### variable TriggeredExit 

```C++
FString UTryllClassifyAttributesLLMParams::TriggeredExit;
```



Exit taken when any attribute triggers, or when on\_failure is Triggered and scoring fails. Empty string = END. Graph exit "triggered" — target node name; empty = END. 


        

<hr>



### variable bOverrideInput 

```C++
bool UTryllClassifyAttributesLLMParams::bOverrideInput;
```



Slot name this node consumes. Empty = "user\_message". Set to true to override the inherited Input value. 


        

<hr>



### variable bOverrideModelName 

```C++
bool UTryllClassifyAttributesLLMParams::bOverrideModelName;
```



Model catalog name. Structural because the token-ID mapping is built from the model's vocabulary at construction. Set to true to override the inherited ModelName value. 


        

<hr>



### variable bOverrideOutputName 

```C++
bool UTryllClassifyAttributesLLMParams::bOverrideOutputName;
```



Slot namespace. Empty = node name. Set to true to override the inherited OutputName value. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllClassifyAttributesLLMParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllClassifyAttributesLLMParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllClassifyAttributesLLMParams.h`

