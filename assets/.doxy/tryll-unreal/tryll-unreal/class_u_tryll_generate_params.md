

# Class UTryllGenerateParams



[**ClassList**](annotated.md) **>** [**UTryllGenerateParams**](class_u_tryll_generate_params.md)



[More...](#detailed-description)

* `#include <TryllGenerateParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  int32 | [**ContextSize**](#variable-contextsize)   = `0`<br> |
|  FString | [**DefaultExit**](#variable-defaultexit)  <br> |
|  FString | [**Grammar**](#variable-grammar)  <br> |
|  ETryllHistoryRole | [**HistoryRole**](#variable-historyrole)   = `ETryllHistoryRole::Assistant`<br> |
|  [**FTryllSlotInput**](struct_f_tryll_slot_input.md) | [**Input**](#variable-input)  <br> |
|  [**FTryllModelName**](struct_f_tryll_model_name.md) | [**ModelName**](#variable-modelname)  <br> |
|  [**FTryllOutputFilterOverrides**](struct_f_tryll_output_filter_overrides.md) | [**OutputFilter**](#variable-outputfilter)  <br> |
|  [**FTryllSlotName**](struct_f_tryll_slot_name.md) | [**OutputName**](#variable-outputname)  <br> |
|  ETryllPlacement | [**Placement**](#variable-placement)   = `ETryllPlacement::BeforeUserAsSystem`<br> |
|  [**FTryllSamplingOverrides**](struct_f_tryll_sampling_overrides.md) | [**Sampling**](#variable-sampling)  <br> |
|  ETryllSendAnswer | [**Send**](#variable-send)   = `ETryllSendAnswer::Streamed`<br> |
|  FString | [**SystemPrompt**](#variable-systemprompt)  <br> |
|  FString | [**Template**](#variable-template)  <br> |
|  bool | [**bOverrideGrammar**](#variable-boverridegrammar)   = `false`<br> |
|  bool | [**bOverrideInput**](#variable-boverrideinput)   = `false`<br> |
|  bool | [**bOverrideModelName**](#variable-boverridemodelname)   = `false`<br> |
|  bool | [**bOverrideOutputName**](#variable-boverrideoutputname)   = `false`<br> |
|  bool | [**bOverrideSystemPrompt**](#variable-boverridesystemprompt)   = `false`<br> |
|  bool | [**bOverrideTemplate**](#variable-boverridetemplate)   = `false`<br> |
|  bool | [**bSubstituteAgentVariables**](#variable-bsubstituteagentvariables)   = `false`<br> |
































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


LLM-driven text generation node. Renders a Mustache template, runs sampling, produces a streamed AnswerText. 


    
## Public Attributes Documentation




### variable ContextSize 

```C++
int32 UTryllGenerateParams::ContextSize;
```



KV-cache / context window (n\_ctx) for this node in tokens. 0 = fall back to the model variant's context\_size, else the server default\_n\_ctx. Validated against the model's trained maximum at agent creation. 


        

<hr>



### variable DefaultExit 

```C++
FString UTryllGenerateParams::DefaultExit;
```



Default exit target — routes here after generation completes. Empty string = END (turn finishes). Graph exit "default" — target node name; empty = END. 


        

<hr>



### variable Grammar 

```C++
FString UTryllGenerateParams::Grammar;
```




<hr>



### variable HistoryRole 

```C++
ETryllHistoryRole UTryllGenerateParams::HistoryRole;
```



Controls whether/how this node's slot replays in later turns' projected transcript. 


        

<hr>



### variable Input 

```C++
FTryllSlotInput UTryllGenerateParams::Input;
```



Upstream inter-node slot this field consumes. Empty = context default. 


        

<hr>



### variable ModelName 

```C++
FTryllModelName UTryllGenerateParams::ModelName;
```




<hr>



### variable OutputFilter 

```C++
FTryllOutputFilterOverrides UTryllGenerateParams::OutputFilter;
```



Opt-in bounded artifact cleanup applied to streamed LLM output before variable substitution, slot/history write, wire delivery, and any downstream consumers. Missing/null = all flags off (defaults). Appended after substitute\_agent\_variables for additive field ids. 


        

<hr>



### variable OutputName 

```C++
FTryllSlotName UTryllGenerateParams::OutputName;
```



Effective output slot name for this node. Empty = the node's own name. 


        

<hr>



### variable Placement 

```C++
ETryllPlacement UTryllGenerateParams::Placement;
```



Where the rendered template body is placed relative to the resolved `input`. 


        

<hr>



### variable Sampling 

```C++
FTryllSamplingOverrides UTryllGenerateParams::Sampling;
```



Sparse sampling overrides applied on top of the model-catalog defaults. Sub-table change fires OnSamplingChanged, which re-resolves overrides against the cached model-default sampling. 


        

<hr>



### variable Send 

```C++
ETryllSendAnswer UTryllGenerateParams::Send;
```



Controls answer-text delivery to the client. Replaces the old `stream: bool`. 


        

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



### variable bOverrideGrammar 

```C++
bool UTryllGenerateParams::bOverrideGrammar;
```



Optional GBNF grammar. When non-empty, output is constrained to this grammar at every decode step (malformed output is unsampleable). Empty = unconstrained. Must contain a `root` rule. Mutable — rebuilt per turn, so a client can flip the node between a strict "command turn" and free chat via ChangeAgentParam. Validated at agent creation (invalid → agent-create failure) and on mutation (invalid → InvalidParamValue 3007). Set to true to override the inherited Grammar value. 


        

<hr>



### variable bOverrideInput 

```C++
bool UTryllGenerateParams::bOverrideInput;
```



Slot name this node consumes as its primary text (the "user turn" of the projected prompt). Empty = "user\_message". Structural: immutable after creation — rebinding would re-wire the slot dataflow that is validated once at agent creation. Set to true to override the inherited Input value. 


        

<hr>



### variable bOverrideModelName 

```C++
bool UTryllGenerateParams::bOverrideModelName;
```



Model catalog name. Empty = use the agent's default\_model\_name. Set to true to override the inherited ModelName value. 


        

<hr>



### variable bOverrideOutputName 

```C++
bool UTryllGenerateParams::bOverrideOutputName;
```



Name this node's output slot is stored under. Empty = the node's own name. Structural: immutable after creation — renaming would re-wire the slot dataflow that is validated once at agent creation. Set to true to override the inherited OutputName value. 


        

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



### variable bSubstituteAgentVariables 

```C++
bool UTryllGenerateParams::bSubstituteAgentVariables;
```



When true, replace `__VARIABLE__` markers in streamed LLM output with the rendered values of agent variables that declared `allow_output_substitution`. Matching is ASCII case-insensitive. Default false. Transformed text is authoritative for the slot, wire answer, history, and any downstream consumers. Appended after default\_exit to keep FlatBuffers field ids additive. 


        

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

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllGenerateParams.h`

