

# Class UTryllSpeakParams



[**ClassList**](annotated.md) **>** [**UTryllSpeakParams**](class_u_tryll_speak_params.md)



[More...](#detailed-description)

* `#include <TryllSpeakParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**DefaultExit**](#variable-defaultexit)  <br> |
|  int32 | [**MinSentenceChars**](#variable-minsentencechars)   = `12`<br> |
|  int32 | [**SpeakerId**](#variable-speakerid)   = `0`<br> |
|  float | [**Speed**](#variable-speed)   = `1.0f`<br> |
|  [**FTryllModelName**](struct_f_tryll_model_name.md) | [**TtsModelName**](#variable-ttsmodelname)  <br> |
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


Standalone TTS-only node. Voices the latest assistant text produced upstream on the current interaction (whatever node wrote it — Generate, CannedResponse, etc.), segments it into sentences, and synthesizes each on a dedicated TTS strand, emitting PCM frames via OnTtsAudio callbacks. Unlike GenerateAndSpeak it runs no LLM inference, so it can voice canned/scripted lines or any upstream-produced answer without paying for a generation pass. 


    
## Public Attributes Documentation




### variable DefaultExit 

```C++
FString UTryllSpeakParams::DefaultExit;
```



Default exit target — routes here after synthesis completes. Empty string = END. Graph exit "default" — target node name; empty = END. 


        

<hr>



### variable MinSentenceChars 

```C++
int32 UTryllSpeakParams::MinSentenceChars;
```



Minimum characters before a sentence boundary fires. Lower values reduce first-utterance latency at the cost of more, smaller TTS calls. 


        

<hr>



### variable SpeakerId 

```C++
int32 UTryllSpeakParams::SpeakerId;
```



TTS speaker / voice index for multi-speaker models (default 0). 


        

<hr>



### variable Speed 

```C++
float UTryllSpeakParams::Speed;
```



TTS speech-rate multiplier. 1.0 = native speed. 


        

<hr>



### variable TtsModelName 

```C++
FTryllModelName UTryllSpeakParams::TtsModelName;
```




<hr>



### variable bOverrideTtsModelName 

```C++
bool UTryllSpeakParams::bOverrideTtsModelName;
```



TTS model catalog name. Optional — when empty the server uses the first available TTS voice in the model catalog. Set it to pick a specific voice. Set to true to override the inherited TtsModelName value. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllSpeakParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllSpeakParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllSpeakParams.h`

