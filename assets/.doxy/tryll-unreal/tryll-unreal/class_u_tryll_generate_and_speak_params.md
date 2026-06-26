

# Class UTryllGenerateAndSpeakParams



[**ClassList**](annotated.md) **>** [**UTryllGenerateAndSpeakParams**](class_u_tryll_generate_and_speak_params.md)



[More...](#detailed-description)

* `#include <TryllGenerateAndSpeakParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**DefaultExit**](#variable-defaultexit)  <br> |
|  int32 | [**MinSentenceChars**](#variable-minsentencechars)   = `12`<br> |
|  [**FTryllModelName**](struct_f_tryll_model_name.md) | [**ModelName**](#variable-modelname)  <br> |
|  ETryllPlacement | [**Placement**](#variable-placement)   = `ETryllPlacement::BeforeUserAsSystem`<br> |
|  [**FTryllSamplingOverrides**](struct_f_tryll_sampling_overrides.md) | [**Sampling**](#variable-sampling)  <br> |
|  int32 | [**SpeakerId**](#variable-speakerid)   = `0`<br> |
|  float | [**Speed**](#variable-speed)   = `1.0f`<br> |
|  FString | [**SystemPrompt**](#variable-systemprompt)  <br> |
|  FString | [**Template**](#variable-template)  <br> |
|  [**FTryllModelName**](struct_f_tryll_model_name.md) | [**TtsModelName**](#variable-ttsmodelname)  <br> |
|  bool | [**bOverrideModelName**](#variable-boverridemodelname)   = `false`<br> |
|  bool | [**bOverrideSystemPrompt**](#variable-boverridesystemprompt)   = `false`<br> |
|  bool | [**bOverrideTemplate**](#variable-boverridetemplate)   = `false`<br> |
|  bool | [**bOverrideTtsModelName**](#variable-boverridettsmodelname)   = `false`<br> |
































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


LLM-driven text generation + streaming TTS synthesis in a single fused node. Streams LLM tokens, segments them into sentences, synthesizes each sentence on a dedicated TTS strand, and emits PCM frames via OnTtsAudio callbacks. 


    
## Public Attributes Documentation




### variable DefaultExit 

```C++
FString UTryllGenerateAndSpeakParams::DefaultExit;
```



Default exit target — routes here after generation + synthesis complete. Empty string = END. Graph exit "default" — target node name; empty = END. 


        

<hr>



### variable MinSentenceChars 

```C++
int32 UTryllGenerateAndSpeakParams::MinSentenceChars;
```



Minimum characters before a sentence boundary fires. Lower values reduce first-utterance latency at the cost of more, smaller TTS calls. 


        

<hr>



### variable ModelName 

```C++
FTryllModelName UTryllGenerateAndSpeakParams::ModelName;
```




<hr>



### variable Placement 

```C++
ETryllPlacement UTryllGenerateAndSpeakParams::Placement;
```



Where the rendered template body is placed in the projected message stream. 


        

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



TTS speaker / voice index for multi-speaker models (default 0). 


        

<hr>



### variable Speed 

```C++
float UTryllGenerateAndSpeakParams::Speed;
```



TTS speech-rate multiplier. 1.0 = native speed. 


        

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



### variable TtsModelName 

```C++
FTryllModelName UTryllGenerateAndSpeakParams::TtsModelName;
```




<hr>



### variable bOverrideModelName 

```C++
bool UTryllGenerateAndSpeakParams::bOverrideModelName;
```



LLM model catalog name. Empty = use the agent's default\_model\_name. Set to true to override the inherited ModelName value. 


        

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



### variable bOverrideTtsModelName 

```C++
bool UTryllGenerateAndSpeakParams::bOverrideTtsModelName;
```



TTS model catalog name. Optional — when empty the server uses the first available TTS voice in the model catalog. Set it to pick a specific voice. Set to true to override the inherited TtsModelName value. 


        

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
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllGenerateAndSpeakParams.h`

