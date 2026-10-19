

# Class UTryllSpeakParams



[**ClassList**](annotated.md) **>** [**UTryllSpeakParams**](class_u_tryll_speak_params.md)



[More...](#detailed-description)

* `#include <TryllSpeakParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**DefaultExit**](#variable-defaultexit)  <br> |
|  int32 | [**FirstChunkMaxChars**](#variable-firstchunkmaxchars)   = `0`<br> |
|  [**FTryllSlotInput**](struct_f_tryll_slot_input.md) | [**Input**](#variable-input)  <br> |
|  int32 | [**MinSentenceChars**](#variable-minsentencechars)   = `12`<br> |
|  bool | [**bOverrideInput**](#variable-boverrideinput)   = `false`<br> |
































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


Standalone TTS-only node. Voices its `input`-resolved slot (any upstream node's output — Generate, CannedResponse, etc. — or `user_message` itself), segments it into sentences, and synthesizes each on a dedicated TTS strand, emitting PCM frames via OnTtsAudio callbacks. Unlike GenerateAndSpeak it runs no LLM inference, so it can voice canned/scripted lines or any upstream-produced answer without paying for a generation pass. The voice belongs to the agent (CreateAgentRequest.voice); agent creation fails without one (3025). 


    
## Public Attributes Documentation




### variable DefaultExit 

```C++
FString UTryllSpeakParams::DefaultExit;
```



Default exit target — routes here after synthesis completes. Empty string = END. Graph exit "default" — target node name; empty = END. 


        

<hr>



### variable FirstChunkMaxChars 

```C++
int32 UTryllSpeakParams::FirstChunkMaxChars;
```



Time-to-first-audio knob. When &gt; 0, the FIRST chunk may also end at a comma, semicolon or colon followed by a space, or at an em dash, and is cut at the next space once it is this wide, so speech starts sooner. Neither cut fires before 5 words or min\_sentence\_chars. Later chunks keep whole sentences. No effect on text without spaces (CJK). 0 = off. Check how it sounds with your voice before shipping it. 


        

<hr>



### variable Input 

```C++
FTryllSlotInput UTryllSpeakParams::Input;
```



Upstream inter-node slot this field consumes. Empty = context default. 


        

<hr>



### variable MinSentenceChars 

```C++
int32 UTryllSpeakParams::MinSentenceChars;
```



Minimum width of the text since the last cut before a sentence end closes a chunk; a CJK or other 3–4-byte character counts as 2. The default 12 is two or three English words. Lower values reduce first-utterance latency at the cost of more, smaller TTS calls. 


        

<hr>



### variable bOverrideInput 

```C++
bool UTryllSpeakParams::bOverrideInput;
```



Slot name this node voices. Empty = "user\_message" (same default as every other input-bearing node). Structural: immutable after creation — rebinding would re-wire the slot dataflow that is validated once at agent creation. Set to true to override the inherited Input value. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllSpeakParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function IsTtsConsumer 

```C++
inline virtual bool UTryllSpeakParams::IsTtsConsumer () override const
```



True when this node speaks through the agent's voice (Speak, GenerateAndSpeak). A graph with such a node needs a voice profile. From the schema's (tts\_consumer). 


        
Implements [*UTryllNodeParamsBase::IsTtsConsumer*](class_u_tryll_node_params_base.md#function-isttsconsumer)


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
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllSpeakParams.h`

