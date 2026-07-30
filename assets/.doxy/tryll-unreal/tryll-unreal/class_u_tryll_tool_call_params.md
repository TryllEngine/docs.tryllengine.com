

# Class UTryllToolCallParams



[**ClassList**](annotated.md) **>** [**UTryllToolCallParams**](class_u_tryll_tool_call_params.md)



[More...](#detailed-description)

* `#include <TryllToolCallParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**AcknowledgeText**](#variable-acknowledgetext)  <br> |
|  int32 | [**ContextSize**](#variable-contextsize)   = `0`<br> |
|  ETryllToolCallDisposition | [**Disposition**](#variable-disposition)   = `ETryllToolCallDisposition::Acknowledge`<br> |
|  [**FTryllSlotInput**](struct_f_tryll_slot_input.md) | [**Input**](#variable-input)  <br> |
|  ETryllToolCallMode | [**Mode**](#variable-mode)   = `ETryllToolCallMode::CallOrAnswer`<br> |
|  [**FTryllModelName**](struct_f_tryll_model_name.md) | [**ModelName**](#variable-modelname)  <br> |
|  FString | [**NoToolCalledExit**](#variable-notoolcalledexit)  <br> |
|  [**FTryllSamplingOverrides**](struct_f_tryll_sampling_overrides.md) | [**Sampling**](#variable-sampling)  <br> |
|  FString | [**SystemPrompt**](#variable-systemprompt)  <br> |
|  FString | [**ToolCallFormat**](#variable-toolcallformat)  <br> |
|  FString | [**ToolCalledExit**](#variable-toolcalledexit)  <br> |
|  TArray&lt; [**FTryllToolDefinition**](struct_f_tryll_tool_definition.md) &gt; | [**Tools**](#variable-tools)  <br> |
|  bool | [**bOverrideAcknowledgeText**](#variable-boverrideacknowledgetext)   = `false`<br> |
|  bool | [**bOverrideInput**](#variable-boverrideinput)   = `false`<br> |
|  bool | [**bOverrideModelName**](#variable-boverridemodelname)   = `false`<br> |
|  bool | [**bOverrideSystemPrompt**](#variable-boverridesystemprompt)   = `false`<br> |
|  bool | [**bOverrideToolCallFormat**](#variable-boverridetoolcallformat)   = `false`<br> |
|  bool | [**bParallelToolCalls**](#variable-bparalleltoolcalls)   = `false`<br> |
































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


LLM-driven tool-call node. Constructs a tool-call prompt, runs sampling, parses the result. 


    
## Public Attributes Documentation




### variable AcknowledgeText 

```C++
FString UTryllToolCallParams::AcknowledgeText;
```




<hr>



### variable ContextSize 

```C++
int32 UTryllToolCallParams::ContextSize;
```



KV-cache / context window (n\_ctx) for this node in tokens. 0 = fall back to the model variant's context\_size, else the server default\_n\_ctx. Validated against the model's trained maximum at agent creation. 


        

<hr>



### variable Disposition 

```C++
ETryllToolCallDisposition UTryllToolCallParams::Disposition;
```



End-to-end notify / pause / history policy for this node. See Tryll.ToolCallDisposition. Mutable: stamped onto ToolCallRecord at write time, so a mid-session change only affects subsequent turns. 


        

<hr>



### variable Input 

```C++
FTryllSlotInput UTryllToolCallParams::Input;
```



Upstream inter-node slot this field consumes. Empty = context default. 


        

<hr>



### variable Mode 

```C++
ETryllToolCallMode UTryllToolCallParams::Mode;
```



Tool-choice policy — see Tryll.ToolCallMode. Mutable: the grammar is rebuilt every turn, so a client can flip a node between (for example) RequireCall for a known-command turn and DetectOnly for free chat. 


        

<hr>



### variable ModelName 

```C++
FTryllModelName UTryllToolCallParams::ModelName;
```




<hr>



### variable NoToolCalledExit 

```C++
FString UTryllToolCallParams::NoToolCalledExit;
```



Exit taken when no tool call is parsed (or, under mode=call\_or\_answer, when the model's residual text was emitted as a plain-text reply instead). Empty string = END. Graph exit "no\_tool\_called" — target node name; empty = END. 


        

<hr>



### variable Sampling 

```C++
FTryllSamplingOverrides UTryllToolCallParams::Sampling;
```



Sparse sampling overrides. 


        

<hr>



### variable SystemPrompt 

```C++
FString UTryllToolCallParams::SystemPrompt;
```




<hr>



### variable ToolCallFormat 

```C++
FString UTryllToolCallParams::ToolCallFormat;
```




<hr>



### variable ToolCalledExit 

```C++
FString UTryllToolCallParams::ToolCalledExit;
```



Exit taken when one or more tool calls are parsed from the response. Empty string = END. Graph exit "tool\_called" — target node name; empty = END. 


        

<hr>



### variable Tools 

```C++
TArray<FTryllToolDefinition> UTryllToolCallParams::Tools;
```



Callable tool definitions. Structural because the tool-call prompt schema and parser contract are materialised at construction. 


        

<hr>



### variable bOverrideAcknowledgeText 

```C++
bool UTryllToolCallParams::bOverrideAcknowledgeText;
```



Result text used when disposition synthesises an acknowledgement (Acknowledge / NotifyAndAcknowledge / PauseAndAcknowledge). Empty = "ok". Ignored for RouteOnly / Notify / Pause / AwaitResult. Set to true to override the inherited AcknowledgeText value. 


        

<hr>



### variable bOverrideInput 

```C++
bool UTryllToolCallParams::bOverrideInput;
```



Slot name this node consumes as its primary text. Empty = "user\_message". Structural: immutable after creation — rebinding would re-wire the slot dataflow that is validated once at agent creation. Set to true to override the inherited Input value. 


        

<hr>



### variable bOverrideModelName 

```C++
bool UTryllToolCallParams::bOverrideModelName;
```



Model catalog name. Empty = use the agent's default\_model\_name. Set to true to override the inherited ModelName value. 


        

<hr>



### variable bOverrideSystemPrompt 

```C++
bool UTryllToolCallParams::bOverrideSystemPrompt;
```



Prepended before the user turn during projection. Set to true to override the inherited SystemPrompt value. 


        

<hr>



### variable bOverrideToolCallFormat 

```C++
bool UTryllToolCallParams::bOverrideToolCallFormat;
```



DEPRECATED (kept for wire/config compatibility; ignored by the server). Tool-call prompting and parsing are now driven by the model's own chat template via llama.cpp common/ (see LlamaCppChatTurn), so no per-dialect format is needed. Structural so mutation is still vetoed. Set to true to override the inherited ToolCallFormat value. 


        

<hr>



### variable bParallelToolCalls 

```C++
bool UTryllToolCallParams::bParallelToolCalls;
```



Allow more than one tool call per turn where the model's chat template supports it (most formats cap at one call when false). Mutable for the same reason as `mode`. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllToolCallParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllToolCallParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllToolCallParams.h`

