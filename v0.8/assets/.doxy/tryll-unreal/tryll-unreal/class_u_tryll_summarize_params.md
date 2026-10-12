

# Class UTryllSummarizeParams



[**ClassList**](annotated.md) **>** [**UTryllSummarizeParams**](class_u_tryll_summarize_params.md)



[More...](#detailed-description)

* `#include <TryllSummarizeParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  [**FTryllContextOverrides**](struct_f_tryll_context_overrides.md) | [**Context**](#variable-context)  <br> |
|  int32 | [**ContextSize**](#variable-contextsize)   = `0`<br> |
|  FString | [**DefaultExit**](#variable-defaultexit)  <br> |
|  int32 | [**KeepRecentTurns**](#variable-keeprecentturns)   = `3`<br> |
|  int32 | [**MaxSummaryTokens**](#variable-maxsummarytokens)   = `256`<br> |
|  [**FTryllModelName**](struct_f_tryll_model_name.md) | [**ModelName**](#variable-modelname)  <br> |
|  [**FTryllSamplingOverrides**](struct_f_tryll_sampling_overrides.md) | [**Sampling**](#variable-sampling)  <br> |
|  FString | [**SystemPrompt**](#variable-systemprompt)  <br> |
|  FString | [**Template**](#variable-template)  <br> |
|  int32 | [**TriggerTokens**](#variable-triggertokens)   = `1024`<br> |
|  int32 | [**TriggerTurns**](#variable-triggerturns)   = `0`<br> |
|  bool | [**bNotifyClient**](#variable-bnotifyclient)   = `false`<br> |
|  bool | [**bOverrideModelName**](#variable-boverridemodelname)   = `false`<br> |
|  bool | [**bOverrideSystemPrompt**](#variable-boverridesystemprompt)   = `false`<br> |
|  bool | [**bOverrideTemplate**](#variable-boverridetemplate)   = `false`<br> |
|  bool | [**bRunInline**](#variable-bruninline)   = `true`<br> |
































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


Rolling conversation summary. Every time the history since the last fold crosses a trigger, this node compresses the older interactions (everything outside `keep_recent_turns`) into a SummaryComponent. Generate-class projection then renders that summary at the end of its system message and replays history only from `covered_through + 1`, so a long dialogue stays coherent at a fixed token cost instead of being trimmed away. Place it last in the graph, after the node that produced the player-facing reply. 


    
## Public Attributes Documentation




### variable Context 

```C++
FTryllContextOverrides UTryllSummarizeParams::Context;
```



Backend context construction (n\_batch / n\_ubatch / kv\_cache\_type / offload\_kqv). Null table inherits; `n_outputs_max` is always forced to 1. 


        

<hr>



### variable ContextSize 

```C++
int32 UTryllSummarizeParams::ContextSize;
```



KV-cache / context window (n\_ctx) for this node's own context, in tokens. 0 = 2048 (the node-type default). Validated against the model's trained maximum at agent creation. 


        

<hr>



### variable DefaultExit 

```C++
FString UTryllSummarizeParams::DefaultExit;
```



Default exit target — routed after the node runs, whether or not it folded. Empty string = END (turn finishes). Graph exit "default" — target node name; empty = END. 


        

<hr>



### variable KeepRecentTurns 

```C++
int32 UTryllSummarizeParams::KeepRecentTurns;
```



Most recent interactions never folded — they always replay verbatim. The summary is always strictly behind this hot window. Capped at 512. 


        

<hr>



### variable MaxSummaryTokens 

```C++
int32 UTryllSummarizeParams::MaxSummaryTokens;
```



Hard cap on the generated summary (applied as the sampling max\_tokens). Must leave at least 64 tokens in context\_size for prompt framing. 


        

<hr>



### variable ModelName 

```C++
FTryllModelName UTryllSummarizeParams::ModelName;
```




<hr>



### variable Sampling 

```C++
FTryllSamplingOverrides UTryllSummarizeParams::Sampling;
```



Sparse sampling overrides applied on top of the model-catalog defaults. 


        

<hr>



### variable SystemPrompt 

```C++
FString UTryllSummarizeParams::SystemPrompt;
```




<hr>



### variable Template 

```C++
FString UTryllSummarizeParams::Template;
```




<hr>



### variable TriggerTokens 

```C++
int32 UTryllSummarizeParams::TriggerTokens;
```



Fold when the not-yet-folded history reaches this many tokens. &lt;= 0 disables this trigger. Set it below the consuming node's trim threshold so a fold always happens before trimming would. 


        

<hr>



### variable TriggerTurns 

```C++
int32 UTryllSummarizeParams::TriggerTurns;
```



Fold every N completed interactions since the last fold. &lt;= 0 disables this trigger. With both triggers disabled the node no-ops. 


        

<hr>



### variable bNotifyClient 

```C++
bool UTryllSummarizeParams::bNotifyClient;
```



When true, fire OnNodeEvent("summarized", …) after a fold. 


        

<hr>



### variable bOverrideModelName 

```C++
bool UTryllSummarizeParams::bOverrideModelName;
```



Language model used for summarization. Empty = the agent's default\_model\_name. The node always runs under its own instruction-only system prompt, never a character persona; point this at a plain instruct model when the character itself runs on roleplay-fine-tuned weights. Set to true to override the inherited ModelName value. 


        

<hr>



### variable bOverrideSystemPrompt 

```C++
bool UTryllSummarizeParams::bOverrideSystemPrompt;
```



Instruction-only system prompt for the summarizer. Empty = the built-in default (third person, named speakers, merge-and-shorten, no speculation). Set to true to override the inherited SystemPrompt value. 


        

<hr>



### variable bOverrideTemplate 

```C++
bool UTryllSummarizeParams::bOverrideTemplate;
```



Mustache body rendered as the summarizer's user turn. Context: `previous_summary` (absent when there is none), `turns` (iterable of {index, user, assistant}); `assistant` joins every assistant-history slot in that interaction in execution order, `max_summary_tokens`, and the session locale keys (`language`, …). Empty = the built-in default. Set to true to override the inherited Template value. 


        

<hr>



### variable bRunInline 

```C++
bool UTryllSummarizeParams::bRunInline;
```



true = fold inside the turn that triggered it (deterministic, costs that turn the summarizer latency). false is reserved for background folding and is rejected until background state publication is implemented. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllSummarizeParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllSummarizeParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllSummarizeParams.h`

