

# Class UTryllVoiceInputComponent



[**ClassList**](annotated.md) **>** [**UTryllVoiceInputComponent**](class_u_tryll_voice_input_component.md)



[More...](#detailed-description)

* `#include <TryllVoiceInputComponent.h>`



Inherits the following classes: UActorComponent


















## Public Attributes

| Type | Name |
| ---: | :--- |
|  float | [**HotwordsScore**](#variable-hotwordsscore)   = `1.5f`<br> |
|  [**FTryllStoragePath**](struct_f_tryll_storage_path.md) | [**HotwordsStoragePath**](#variable-hotwordsstoragepath)  <br> |
|  int32 | [**MaxUtteranceMs**](#variable-maxutterancems)   = `60000`<br> |
|  [**FTryllModelName**](struct_f_tryll_model_name.md) | [**ModelName**](#variable-modelname)   = `TEXT("Whisper Tiny EN (int8)")`<br> |
|  FOnTryllVoiceError | [**OnError**](#variable-onerror)  <br> |
|  FOnTryllTranscriptUpdate | [**OnTranscriptUpdate**](#variable-ontranscriptupdate)  <br> |
|  FOnTryllVoiceInputCreated | [**OnVoiceInputCreated**](#variable-onvoiceinputcreated)  <br> |
|  [**UTryllAgentComponent**](class_u_tryll_agent_component.md) \* | [**TargetAgent**](#variable-targetagent)   = `nullptr`<br> |
|  int32 | [**VadMinSilenceMs**](#variable-vadminsilencems)   = `500`<br> |
|  int32 | [**VadSpeechPadMs**](#variable-vadspeechpadms)   = `250`<br> |
|  float | [**VadThreshold**](#variable-vadthreshold)   = `0.5f`<br> |
|  bool | [**bAutoFinishOnSilence**](#variable-bautofinishonsilence)   = `true`<br> |
|  bool | [**bCreateOnConnect**](#variable-bcreateonconnect)   = `true`<br> |
|  bool | [**bWaitForAgentReady**](#variable-bwaitforagentready)   = `true`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  void | [**BeginUtterance**](#function-beginutterance) ([**UTryllAgentComponent**](class_u_tryll_agent_component.md) \* AgentOverride=nullptr) <br> |
|  void | [**CancelUtterance**](#function-cancelutterance) () <br> |
|  void | [**CreateVoiceInput**](#function-createvoiceinput) () <br> |
|  void | [**DestroyVoiceInput**](#function-destroyvoiceinput) () <br> |
|  void | [**EndUtterance**](#function-endutterance) () <br> |
|  TSharedPtr&lt; [**FTryllVoiceInput**](class_f_tryll_voice_input.md) &gt; | [**GetVoiceInput**](#function-getvoiceinput) () const<br> |
|  bool | [**HasVoiceInput**](#function-hasvoiceinput) () const<br> |
|  bool | [**IsUtteranceActive**](#function-isutteranceactive) () const<br> |
|   | [**UTryllVoiceInputComponent**](#function-utryllvoiceinputcomponent) () <br> |
























## Protected Functions

| Type | Name |
| ---: | :--- |
| virtual void | [**BeginPlay**](#function-beginplay) () override<br> |
| virtual void | [**EndPlay**](#function-endplay) (const EEndPlayReason::Type EndPlayReason) override<br> |
| virtual void | [**TickComponent**](#function-tickcomponent) (float DeltaTime, ELevelTick TickType, FActorComponentTickFunction \* ThisTickFunction) override<br> |




## Detailed Description


Actor component: owns one [**FTryllVoiceInput**](class_f_tryll_voice_input.md) (STT) handle and drives it from the system microphone. The Unreal counterpart to the Unity TryllVoiceInputComponent.


Capture model: on-demand. The microphone opens on [**BeginUtterance()**](class_u_tryll_voice_input_component.md#function-beginutterance) and closes when the final transcript arrives (or on End / Cancel). The component ticks to drain captured PCM and forward it to the server.


Audio: the device-native sample rate is declared to the server, which resamples to the STT model's expected rate. Multi-channel input is downmixed to mono.


All delegates fire on the game thread. 


    
## Public Attributes Documentation




### variable HotwordsScore 

```C++
float UTryllVoiceInputComponent::HotwordsScore;
```



Logit-bias score applied to hotword tokens during decoding. 


        

<hr>



### variable HotwordsStoragePath 

```C++
FTryllStoragePath UTryllVoiceInputComponent::HotwordsStoragePath;
```



Plain string storage containing hotword / wake-word phrases (one per line). Relative to TryllRuntimeSettings::StorageDataFolder, e.g. "hotwords.txt". Leave empty to disable hotword boosting. 


        

<hr>



### variable MaxUtteranceMs 

```C++
int32 UTryllVoiceInputComponent::MaxUtteranceMs;
```



Hard timeout (ms) per utterance. 


        

<hr>



### variable ModelName 

```C++
FTryllModelName UTryllVoiceInputComponent::ModelName;
```



STT model — picked from the project's registered models (Model Manager). 


        

<hr>



### variable OnError 

```C++
FOnTryllVoiceError UTryllVoiceInputComponent::OnError;
```



Fired on protocol-level errors. 


        

<hr>



### variable OnTranscriptUpdate 

```C++
FOnTryllTranscriptUpdate UTryllVoiceInputComponent::OnTranscriptUpdate;
```



Fired for each transcript update (partial and final). 


        

<hr>



### variable OnVoiceInputCreated 

```C++
FOnTryllVoiceInputCreated UTryllVoiceInputComponent::OnVoiceInputCreated;
```



Fired when the VoiceInput handle has been created. 


        

<hr>



### variable TargetAgent 

```C++
UTryllAgentComponent* UTryllVoiceInputComponent::TargetAgent;
```



Agent that receives the final transcript. Leave unset to auto-use the [**UTryllAgentComponent**](class_u_tryll_agent_component.md) on the same actor (the common case — the details panel can't pick sibling components, so this resolves in BeginPlay). Put the Voice component on an actor with no agent for transcribe-only mode. 


        

<hr>



### variable VadMinSilenceMs 

```C++
int32 UTryllVoiceInputComponent::VadMinSilenceMs;
```



Silence duration (ms) that closes a speech segment. 


        

<hr>



### variable VadSpeechPadMs 

```C++
int32 UTryllVoiceInputComponent::VadSpeechPadMs;
```



Padding (ms) added around detected speech. 


        

<hr>



### variable VadThreshold 

```C++
float UTryllVoiceInputComponent::VadThreshold;
```



Silero VAD speech-probability threshold. 


        

<hr>



### variable bAutoFinishOnSilence 

```C++
bool UTryllVoiceInputComponent::bAutoFinishOnSilence;
```



Let server VAD close the utterance on silence (stop-on-silence mode). 


        

<hr>



### variable bCreateOnConnect 

```C++
bool UTryllVoiceInputComponent::bCreateOnConnect;
```



Create the VoiceInput handle automatically once the session is ready (connected + configured); queues until OnConfigureSessionComplete if it spawns earlier. (Name kept for compatibility; it fires on session-ready, not raw connect.) 


        

<hr>



### variable bWaitForAgentReady 

```C++
bool UTryllVoiceInputComponent::bWaitForAgentReady;
```



Sequence VoiceInput creation behind the sibling agent: wait for the agent's OnAgentReady before creating the VoiceInput, instead of creating as soon as the session is ready. This serialises first-run model auto-downloads so the STT/VAD fetch never overlaps the agent's LLM/TTS fetch — concurrent downloads deadlock the server's shared HuggingFace connection. Ignored when there is no TargetAgent (transcribe-only). Set false to restore the old concurrent behaviour. 


        

<hr>
## Public Functions Documentation




### function BeginUtterance 

```C++
void UTryllVoiceInputComponent::BeginUtterance (
    UTryllAgentComponent * AgentOverride=nullptr
) 
```



Open a new utterance and start microphone capture. The final transcript is routed to AgentOverride when given, otherwise to TargetAgent; transcribe-only when both are null. 


        

<hr>



### function CancelUtterance 

```C++
void UTryllVoiceInputComponent::CancelUtterance () 
```



Cancel the current utterance without producing a transcript. 


        

<hr>



### function CreateVoiceInput 

```C++
void UTryllVoiceInputComponent::CreateVoiceInput () 
```



Create the server-side VoiceInput handle. Ignored when one already exists. 


        

<hr>



### function DestroyVoiceInput 

```C++
void UTryllVoiceInputComponent::DestroyVoiceInput () 
```



Destroy the handle and release the server-side session. 


        

<hr>



### function EndUtterance 

```C++
void UTryllVoiceInputComponent::EndUtterance () 
```



Commit the current utterance (push-to-talk release). 


        

<hr>



### function GetVoiceInput 

```C++
inline TSharedPtr< FTryllVoiceInput > UTryllVoiceInputComponent::GetVoiceInput () const
```



Live handle. Null until CreateVoiceInput completes. 


        

<hr>



### function HasVoiceInput 

```C++
bool UTryllVoiceInputComponent::HasVoiceInput () const
```



True once the server-side VoiceInput handle exists. 


        

<hr>



### function IsUtteranceActive 

```C++
bool UTryllVoiceInputComponent::IsUtteranceActive () const
```



True while an utterance is open. 


        

<hr>



### function UTryllVoiceInputComponent 

```C++
UTryllVoiceInputComponent::UTryllVoiceInputComponent () 
```




<hr>
## Protected Functions Documentation




### function BeginPlay 

```C++
virtual void UTryllVoiceInputComponent::BeginPlay () override
```




<hr>



### function EndPlay 

```C++
virtual void UTryllVoiceInputComponent::EndPlay (
    const EEndPlayReason::Type EndPlayReason
) override
```




<hr>



### function TickComponent 

```C++
virtual void UTryllVoiceInputComponent::TickComponent (
    float DeltaTime,
    ELevelTick TickType,
    FActorComponentTickFunction * ThisTickFunction
) override
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllVoiceInputComponent.h`

