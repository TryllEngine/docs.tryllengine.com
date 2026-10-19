

# Class UTryllGenerateAndSpeakParams



[**ClassList**](annotated.md) **>** [**UTryllGenerateAndSpeakParams**](class_u_tryll_generate_and_speak_params.md)



[More...](#detailed-description)

* `#include <TryllGenerateAndSpeakParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  [**FTryllModelAdaptation**](struct_f_tryll_model_adaptation.md) | [**Adaptation**](#variable-adaptation)  <br> |
|  [**FTryllContextOverrides**](struct_f_tryll_context_overrides.md) | [**Context**](#variable-context)  <br> |
|  int32 | [**ContextSize**](#variable-contextsize)   = `0`<br> |
|  FString | [**DefaultExit**](#variable-defaultexit)  <br> |
|  int32 | [**FirstChunkMaxChars**](#variable-firstchunkmaxchars)   = `0`<br> |
|  FString | [**Grammar**](#variable-grammar)  <br> |
|  ETryllHistoryRole | [**HistoryRole**](#variable-historyrole)   = `ETryllHistoryRole::Assistant`<br> |
|  [**FTryllSlotInput**](struct_f_tryll_slot_input.md) | [**Input**](#variable-input)  <br> |
|  int32 | [**MinSentenceChars**](#variable-minsentencechars)   = `12`<br> |
|  [**FTryllModelName**](struct_f_tryll_model_name.md) | [**ModelName**](#variable-modelname)  <br> |
|  [**FTryllOutputFilterOverrides**](struct_f_tryll_output_filter_overrides.md) | [**OutputFilter**](#variable-outputfilter)  <br> |
|  [**FTryllSlotName**](struct_f_tryll_slot_name.md) | [**OutputName**](#variable-outputname)  <br> |
|  ETryllPlacement | [**Placement**](#variable-placement)   = `ETryllPlacement::BeforeUserAsSystem`<br> |
|  [**FTryllSamplingOverrides**](struct_f_tryll_sampling_overrides.md) | [**Sampling**](#variable-sampling)  <br> |
|  FString | [**SystemPrompt**](#variable-systemprompt)  <br> |
|  FString | [**SystemPromptTextId**](#variable-systemprompttextid)  <br> |
|  FString | [**Template**](#variable-template)  <br> |
|  FString | [**TemplateTextId**](#variable-templatetextid)  <br> |
|  bool | [**bOverrideGrammar**](#variable-boverridegrammar)   = `false`<br> |
|  bool | [**bOverrideInput**](#variable-boverrideinput)   = `false`<br> |
|  bool | [**bOverrideModelName**](#variable-boverridemodelname)   = `false`<br> |
|  bool | [**bOverrideOutputName**](#variable-boverrideoutputname)   = `false`<br> |
|  bool | [**bOverrideSystemPrompt**](#variable-boverridesystemprompt)   = `false`<br> |
|  bool | [**bOverrideTemplate**](#variable-boverridetemplate)   = `false`<br> |
|  bool | [**bSubstituteAgentVariables**](#variable-bsubstituteagentvariables)   = `false`<br> |
|  bool | [**bSystemPromptLocalized**](#variable-bsystempromptlocalized)   = `false`<br> |
|  bool | [**bTemplateLocalized**](#variable-btemplatelocalized)   = `false`<br> |
































## Public Functions

| Type | Name |
| ---: | :--- |
| virtual ETryllNodeType | [**GetNodeType**](#function-getnodetype) () override const<br> |
| virtual bool | [**IsTtsConsumer**](#function-isttsconsumer) () override const<br> |
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


LLM-driven text generation + streaming TTS synthesis in a single fused node. Streams LLM tokens, segments them into sentences, synthesizes each sentence on a dedicated TTS strand, and emits PCM frames via OnTtsAudio callbacks. The voice (model, speaker, delivery mode, visemes) belongs to the agent: CreateAgentRequest.voice. Agent creation fails without one (3025). 


    
## Public Attributes Documentation




### variable Adaptation 

```C++
FTryllModelAdaptation UTryllGenerateAndSpeakParams::Adaptation;
```



Changes how the model behaves for this node (LoRA adapters; later control vectors). Null = base model. Structural: applied to the llama context at creation, so a change means a new context. Appended last so existing field IDs do not move. 


        

<hr>



### variable Context 

```C++
FTryllContextOverrides UTryllGenerateAndSpeakParams::Context;
```



Backend context construction. Null table inherits, except CreateContext still defaults `n_outputs_max` to 1. Structural. 


        

<hr>



### variable ContextSize 

```C++
int32 UTryllGenerateAndSpeakParams::ContextSize;
```



KV-cache / context window (n\_ctx) for the LLM in tokens. 0 = fall back to the model variant's context\_size, else the server default\_n\_ctx. Validated against the model's trained maximum at agent creation. 


        

<hr>



### variable DefaultExit 

```C++
FString UTryllGenerateAndSpeakParams::DefaultExit;
```



Default exit target — routes here after generation + synthesis complete. Empty string = END. Graph exit "default" — target node name; empty = END. 


        

<hr>



### variable FirstChunkMaxChars 

```C++
int32 UTryllGenerateAndSpeakParams::FirstChunkMaxChars;
```



Time-to-first-audio knob. When &gt; 0, the FIRST chunk may also end at a comma, semicolon or colon followed by a space, or at an em dash, and is cut at the next space once it is this wide, so speech starts sooner. Neither cut fires before 5 words or min\_sentence\_chars. Later chunks keep whole sentences. No effect on text without spaces (CJK). 0 = off. 48 is the measured starting point: about 40% less model-side wait on Supertonic and Pocket. Check how it sounds with your voice before shipping it. 


        

<hr>



### variable Grammar 

```C++
FString UTryllGenerateAndSpeakParams::Grammar;
```




<hr>



### variable HistoryRole 

```C++
ETryllHistoryRole UTryllGenerateAndSpeakParams::HistoryRole;
```



Controls whether/how this node's slot replays in later turns' projected transcript. No `send` field — GenerateAndSpeak always streams text (streaming and non-streaming delivery would need incompatible projection-replay handling, so only the streamed mode is supported). 


        

<hr>



### variable Input 

```C++
FTryllSlotInput UTryllGenerateAndSpeakParams::Input;
```



Upstream inter-node slot this field consumes. Empty = context default. 


        

<hr>



### variable MinSentenceChars 

```C++
int32 UTryllGenerateAndSpeakParams::MinSentenceChars;
```



Minimum width of the text since the last cut before a sentence end closes a chunk; a CJK or other 3–4-byte character counts as 2. The default 12 is two or three English words. Lower values reduce first-utterance latency at the cost of more, smaller TTS calls. 


        

<hr>



### variable ModelName 

```C++
FTryllModelName UTryllGenerateAndSpeakParams::ModelName;
```




<hr>



### variable OutputFilter 

```C++
FTryllOutputFilterOverrides UTryllGenerateAndSpeakParams::OutputFilter;
```



Opt-in bounded artifact cleanup applied to streamed LLM output before variable substitution, slot/history write, wire delivery, and TTS. Missing/null = all flags off (defaults). 


        

<hr>



### variable OutputName 

```C++
FTryllSlotName UTryllGenerateAndSpeakParams::OutputName;
```



Effective output slot name for this node. Empty = the node's own name. 


        

<hr>



### variable Placement 

```C++
ETryllPlacement UTryllGenerateAndSpeakParams::Placement;
```



Where the rendered template body is placed relative to the resolved `input`. 


        

<hr>



### variable Sampling 

```C++
FTryllSamplingOverrides UTryllGenerateAndSpeakParams::Sampling;
```



Sparse sampling overrides applied on top of the model-catalog defaults. 


        

<hr>



### variable SystemPrompt 

```C++
FString UTryllGenerateAndSpeakParams::SystemPrompt;
```




<hr>



### variable SystemPromptTextId 

```C++
FString UTryllGenerateAndSpeakParams::SystemPromptTextId;
```



Text ID in the workflow text table. The provider's text is what is sent. 


        

<hr>



### variable Template 

```C++
FString UTryllGenerateAndSpeakParams::Template;
```




<hr>



### variable TemplateTextId 

```C++
FString UTryllGenerateAndSpeakParams::TemplateTextId;
```



Text ID in the workflow text table. The provider's text is what is sent. 


        

<hr>



### variable bOverrideGrammar 

```C++
bool UTryllGenerateAndSpeakParams::bOverrideGrammar;
```



Optional GBNF grammar. When non-empty, the generated (and spoken) output is constrained to this grammar at every decode step. Empty = unconstrained. Must contain a `root` rule. Mutable — rebuilt per turn. Narrower fit than on Generate: use it to constrain spoken output to a fixed set of barks / canned lines. Validated at agent creation and on mutation (3007). Set to true to override the inherited Grammar value. 


        

<hr>



### variable bOverrideInput 

```C++
bool UTryllGenerateAndSpeakParams::bOverrideInput;
```



Slot name this node consumes as its primary text. Empty = "user\_message". Structural: immutable after creation — rebinding would re-wire the slot dataflow that is validated once at agent creation. Set to true to override the inherited Input value. 


        

<hr>



### variable bOverrideModelName 

```C++
bool UTryllGenerateAndSpeakParams::bOverrideModelName;
```



LLM model catalog name. Empty = use the agent's default\_model\_name. Set to true to override the inherited ModelName value. 


        

<hr>



### variable bOverrideOutputName 

```C++
bool UTryllGenerateAndSpeakParams::bOverrideOutputName;
```



Name this node's output slot is stored under. Empty = the node's own name. Structural: immutable after creation — renaming would re-wire the slot dataflow that is validated once at agent creation. Set to true to override the inherited OutputName value. 


        

<hr>



### variable bOverrideSystemPrompt 

```C++
bool UTryllGenerateAndSpeakParams::bOverrideSystemPrompt;
```



Prepended before the user turn during projection. Empty = none. Set to true to override the inherited SystemPrompt value. 


        

<hr>



### variable bOverrideTemplate 

```C++
bool UTryllGenerateAndSpeakParams::bOverrideTemplate;
```



Mustache template applied to the user-area message at projection time. Empty = no template. Set to true to override the inherited Template value. 


        

<hr>



### variable bSubstituteAgentVariables 

```C++
bool UTryllGenerateAndSpeakParams::bSubstituteAgentVariables;
```



When true, replace `__VARIABLE__` markers in streamed LLM output with the rendered values of agent variables that declared `allow_output_substitution`. Matching is ASCII case-insensitive. Default false. Transformed text is authoritative for the slot, wire answer, history, and TTS input (substitution runs before synthesis). 


        

<hr>



### variable bSystemPromptLocalized 

```C++
bool UTryllGenerateAndSpeakParams::bSystemPromptLocalized;
```



Ticked: SystemPromptTextId is resolved by the text provider and that text is sent. 


        

<hr>



### variable bTemplateLocalized 

```C++
bool UTryllGenerateAndSpeakParams::bTemplateLocalized;
```



Ticked: TemplateTextId is resolved by the text provider and that text is sent. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllGenerateAndSpeakParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function IsTtsConsumer 

```C++
inline virtual bool UTryllGenerateAndSpeakParams::IsTtsConsumer () override const
```



True when this node speaks through the agent's voice (Speak, GenerateAndSpeak). A graph with such a node needs a voice profile. From the schema's (tts\_consumer). 


        
Implements [*UTryllNodeParamsBase::IsTtsConsumer*](class_u_tryll_node_params_base.md#function-isttsconsumer)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllGenerateAndSpeakParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllGenerateAndSpeakParams.h`

