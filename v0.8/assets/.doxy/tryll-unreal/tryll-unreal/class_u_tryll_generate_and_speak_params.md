

# Class UTryllGenerateAndSpeakParams



[**ClassList**](annotated.md) **>** [**UTryllGenerateAndSpeakParams**](class_u_tryll_generate_and_speak_params.md)



[More...](#detailed-description)

* `#include <TryllGenerateAndSpeakParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
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
|  int32 | [**SpeakerId**](#variable-speakerid)   = `0`<br> |
|  float | [**Speed**](#variable-speed)   = `1.0f`<br> |
|  FString | [**SystemPrompt**](#variable-systemprompt)  <br> |
|  FString | [**Template**](#variable-template)  <br> |
|  FString | [**TtsDeliveryInstruction**](#variable-ttsdeliveryinstruction)  <br> |
|  FString | [**TtsLang**](#variable-ttslang)  <br> |
|  [**FTryllModelName**](struct_f_tryll_model_name.md) | [**TtsModelName**](#variable-ttsmodelname)  <br> |
|  [**FTryllStoragePath**](struct_f_tryll_storage_path.md) | [**TtsVoice**](#variable-ttsvoice)  <br> |
|  FString | [**TtsVoiceDescription**](#variable-ttsvoicedescription)  <br> |
|  bool | [**bEmitVisemes**](#variable-bemitvisemes)   = `false`<br> |
|  bool | [**bOverrideGrammar**](#variable-boverridegrammar)   = `false`<br> |
|  bool | [**bOverrideInput**](#variable-boverrideinput)   = `false`<br> |
|  bool | [**bOverrideModelName**](#variable-boverridemodelname)   = `false`<br> |
|  bool | [**bOverrideOutputName**](#variable-boverrideoutputname)   = `false`<br> |
|  bool | [**bOverrideSystemPrompt**](#variable-boverridesystemprompt)   = `false`<br> |
|  bool | [**bOverrideTemplate**](#variable-boverridetemplate)   = `false`<br> |
|  bool | [**bOverrideTtsDeliveryInstruction**](#variable-boverridettsdeliveryinstruction)   = `false`<br> |
|  bool | [**bOverrideTtsLang**](#variable-boverridettslang)   = `false`<br> |
|  bool | [**bOverrideTtsModelName**](#variable-boverridettsmodelname)   = `false`<br> |
|  bool | [**bOverrideTtsVoiceDescription**](#variable-boverridettsvoicedescription)   = `false`<br> |
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


LLM-driven text generation + streaming TTS synthesis in a single fused node. Streams LLM tokens, segments them into sentences, synthesizes each sentence on a dedicated TTS strand, and emits PCM frames via OnTtsAudio callbacks. 


    
## Public Attributes Documentation




### variable Context 

```C++
FTryllContextOverrides UTryllGenerateAndSpeakParams::Context;
```



Backend context construction. Null table inherits, except CreateContext still defaults `n_outputs_max` to 1. Structural. Appended after tts\_delivery\_instruction so existing field IDs do not move. 


        

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



Time-to-first-audio knob. When &gt; 0, the FIRST chunk may also end at a comma, semicolon or colon followed by a space, or at an em dash, and is cut at the next space once it is this wide, so speech starts sooner. Neither cut fires before 5 words or min\_sentence\_chars. Later chunks keep whole sentences. No effect on text without spaces (CJK). 0 = off. 48 is the measured starting point: about 40% less model-side wait on Supertonic and Pocket. Check how it sounds with your voice before shipping it. Appended after emit\_visemes so existing field IDs do not move. 


        

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



Opt-in bounded artifact cleanup applied to streamed LLM output before variable substitution, slot/history write, wire delivery, and TTS. Missing/null = all flags off (defaults). Appended after substitute\_agent\_variables for additive field ids. 


        

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



### variable SpeakerId 

```C++
int32 UTryllGenerateAndSpeakParams::SpeakerId;
```



TTS speaker / voice index (default 0). Valid range is 0 to the model's speaker count minus 1 — the shipped Supertonic 3 bundle has 10 voice styles (0–9); Pocket TTS is single-speaker, so it must stay 0 there (pick a Pocket voice with tts\_voice instead). Agent creation fails with an out-of-range error for values the model cannot satisfy. 


        

<hr>



### variable Speed 

```C++
float UTryllGenerateAndSpeakParams::Speed;
```



TTS speech-rate multiplier. 1.0 = native speed. Supertonic 3 and Pocket TTS accept 0.1–3.0; Orpheus and Maya1 accept only 1.0 (anything else fails agent creation, or a later change, with an unsupported-control error). 


        

<hr>



### variable SystemPrompt 

```C++
FString UTryllGenerateAndSpeakParams::SystemPrompt;
```




<hr>



### variable Template 

```C++
FString UTryllGenerateAndSpeakParams::Template;
```




<hr>



### variable TtsDeliveryInstruction 

```C++
FString UTryllGenerateAndSpeakParams::TtsDeliveryInstruction;
```




<hr>



### variable TtsLang 

```C++
FString UTryllGenerateAndSpeakParams::TtsLang;
```




<hr>



### variable TtsModelName 

```C++
FTryllModelName UTryllGenerateAndSpeakParams::TtsModelName;
```




<hr>



### variable TtsVoice 

```C++
FTryllStoragePath UTryllGenerateAndSpeakParams::TtsVoice;
```



Reference-voice WAV for zero-shot voice-cloning families (Pocket TTS): a path relative to the process storage root pointing at a 10–20 s clean mono recording of the target voice. Empty = the model's default voice. Rejected on models without reference-voice support (every family except Pocket TTS — use speaker\_id there). The file must exist at agent creation. 


        

<hr>



### variable TtsVoiceDescription 

```C++
FString UTryllGenerateAndSpeakParams::TtsVoiceDescription;
```




<hr>



### variable bEmitVisemes 

```C++
bool UTryllGenerateAndSpeakParams::bEmitVisemes;
```



When true, also send a lip-sync viseme track: one `TtsVisemeFrame` per synthesized sentence with the mouth shapes (15 Oculus visemes) and their start times, in samples from the start of this node's audio. See the "Add Lip Sync to a Character" how-to. Mutable. Appended after context so existing FlatBuffers field IDs do not move. 


        

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



Prepended before the user turn during projection. Set to true to override the inherited SystemPrompt value. 


        

<hr>



### variable bOverrideTemplate 

```C++
bool UTryllGenerateAndSpeakParams::bOverrideTemplate;
```



Mustache template applied to the user-area message at projection time. Set to true to override the inherited Template value. 


        

<hr>



### variable bOverrideTtsDeliveryInstruction 

```C++
bool UTryllGenerateAndSpeakParams::bOverrideTtsDeliveryInstruction;
```



Natural-language sustained delivery direction for this synthesis call. Empty = neutral / model default. Maya1 only, with the same limits as tts\_voice\_description. Set to true to override the inherited TtsDeliveryInstruction value. 


        

<hr>



### variable bOverrideTtsLang 

```C++
bool UTryllGenerateAndSpeakParams::bOverrideTtsLang;
```



TTS language hint for multilingual models that select the language per call (e.g. Supertonic 3: "en", "de", "fr", …). Resolution order: this param, then the session locale (CreateSessionRequest.locale), then the model's catalog default. Set it only to override the session — one character who keeps speaking French in a German playthrough. For a whole-game language switch, set the session locale instead and leave this empty: it reaches every agent and every node with no per-node mutation. Ignored by families whose language is fixed by the model itself (e.g. Pocket TTS per-language models). Set to true to override the inherited TtsLang value. 


        

<hr>



### variable bOverrideTtsModelName 

```C++
bool UTryllGenerateAndSpeakParams::bOverrideTtsModelName;
```



TTS model catalog name. Optional — when empty the server uses the first TTS entry in the model catalog, whether or not it is downloaded. Set it to pick a specific voice. Set to true to override the inherited TtsModelName value. 


        

<hr>



### variable bOverrideTtsVoiceDescription 

```C++
bool UTryllGenerateAndSpeakParams::bOverrideTtsVoiceDescription;
```



Natural-language stable voice identity: age, accent, pitch, timbre and baseline pacing. Empty = use speaker\_id / catalog default. Maya1 only: on another voice it fails agent creation. At most 1024 bytes, with no `"`, `<`, `>` or control characters. Appended after output\_filter so existing FlatBuffers field IDs do not move. Set to true to override the inherited TtsVoiceDescription value. 


        

<hr>



### variable bSubstituteAgentVariables 

```C++
bool UTryllGenerateAndSpeakParams::bSubstituteAgentVariables;
```



When true, replace `__VARIABLE__` markers in streamed LLM output with the rendered values of agent variables that declared `allow_output_substitution`. Matching is ASCII case-insensitive. Default false. Transformed text is authoritative for the slot, wire answer, history, and TTS input (substitution runs before synthesis). Appended after default\_exit to keep FlatBuffers field ids additive. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllGenerateAndSpeakParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


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

