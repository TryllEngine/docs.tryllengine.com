

# Class FTryllVoiceInput



[**ClassList**](annotated.md) **>** [**FTryllVoiceInput**](class_f_tryll_voice_input.md)



[More...](#detailed-description)

* `#include <TryllVoiceInput.h>`



Inherits the following classes: TSharedFromThis< FTryllVoiceInput >


































## Public Functions

| Type | Name |
| ---: | :--- |
|  void | [**BeginUtterance**](#function-beginutterance) (const [**FTryllUtteranceOptions**](struct_f_tryll_utterance_options.md) & Opts, TFunction&lt; void(const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete=nullptr) <br> |
|  void | [**CancelUtterance**](#function-cancelutterance) (TFunction&lt; void(const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete=nullptr) <br> |
|  void | [**ClearAfterDestroyRequest**](#function-clearafterdestroyrequest) () <br> |
|  void | [**EndUtterance**](#function-endutterance) (TFunction&lt; void(const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete=nullptr) <br> |
|   | [**FTryllVoiceInput**](#function-ftryllvoiceinput) (TSharedPtr&lt; FTryllConnection &gt; InConnection, std::uint64\_t InVoiceInputId) <br> |
|  std::uint64\_t | [**GetVoiceInputId**](#function-getvoiceinputid) () noexcept const<br> |
|  void | [**HandleAckResult**](#function-handleackresult) (std::uint64\_t RequestId, const [**FTryllError**](struct_f_tryll_error.md) & Error) <br> |
|  void | [**HandleError**](#function-handleerror) (const [**FTryllError**](struct_f_tryll_error.md) & Error) <br> |
|  void | [**HandleTranscriptUpdate**](#function-handletranscriptupdate) (const FString & Text, ETryllTranscriptUpdateKind Kind, int32 AudioMsConsumed) <br> |
|  bool | [**IsUtteranceActive**](#function-isutteranceactive) () noexcept const<br> |
|  bool | [**IsValid**](#function-isvalid) () noexcept const<br> |
|  void | [**SendAudioBuffer**](#function-sendaudiobuffer-12) (const int16 \* Samples, int32 NumSamples) <br> |
|  void | [**SendAudioBuffer**](#function-sendaudiobuffer-22) (const TArray&lt; int16 &gt; & Samples) <br> |
|  void | [**SetOnError**](#function-setonerror) (TFunction&lt; void(const [**FTryllError**](struct_f_tryll_error.md) &)&gt; Callback) <br> |
|  void | [**SetOnTranscriptUpdate**](#function-setontranscriptupdate) (TFunction&lt; void(const [**FTryllTranscriptUpdate**](struct_f_tryll_transcript_update.md) &)&gt; Callback) <br> |
|   | [**~FTryllVoiceInput**](#function-ftryllvoiceinput) () <br> |




























## Detailed Description


Plain C++ handle for a server-side VoiceInput (STT) session. Created by [**UTryllSubsystem**](class_u_tryll_subsystem.md) after a successful CreateVoiceInputResponse. The Unreal counterpart to the C++ Tryll::VoiceInput and the Unity TryllVoiceInput.


Ownership: typically held as TSharedPtr&lt;FTryllVoiceInput&gt; by [**UTryllVoiceInputComponent**](class_u_tryll_voice_input_component.md). Lifetime: deterministic — the owning component releases the shared\_ptr; this triggers [**UTryllSubsystem::RequestDestroyVoiceInput()**](class_u_tryll_subsystem.md#function-requestdestroyvoiceinput).


All callbacks are invoked on the game thread (from UTryllSubsystem::Tick). 


    
## Public Functions Documentation




### function BeginUtterance 

```C++
void FTryllVoiceInput::BeginUtterance (
    const FTryllUtteranceOptions & Opts,
    TFunction< void(const FTryllError &)> OnComplete=nullptr
) 
```



Open a new utterance. Rejects locally (via OnComplete) with UtteranceInProgress when one is already active, or VoiceInputNotFound when the handle is invalid. 


        

<hr>



### function CancelUtterance 

```C++
void FTryllVoiceInput::CancelUtterance (
    TFunction< void(const FTryllError &)> OnComplete=nullptr
) 
```



Cancel the current utterance without producing a transcript. 


        

<hr>



### function ClearAfterDestroyRequest 

```C++
void FTryllVoiceInput::ClearAfterDestroyRequest () 
```



Called by [**UTryllSubsystem**](class_u_tryll_subsystem.md) when the DestroyVoiceInput Ack arrives; clears the local id. 


        

<hr>



### function EndUtterance 

```C++
void FTryllVoiceInput::EndUtterance (
    TFunction< void(const FTryllError &)> OnComplete=nullptr
) 
```



Commit the current utterance (push-to-talk release). 


        

<hr>



### function FTryllVoiceInput 

```C++
FTryllVoiceInput::FTryllVoiceInput (
    TSharedPtr< FTryllConnection > InConnection,
    std::uint64_t InVoiceInputId
) 
```




<hr>



### function GetVoiceInputId 

```C++
inline std::uint64_t FTryllVoiceInput::GetVoiceInputId () noexcept const
```




<hr>



### function HandleAckResult 

```C++
void FTryllVoiceInput::HandleAckResult (
    std::uint64_t RequestId,
    const FTryllError & Error
) 
```




<hr>



### function HandleError 

```C++
void FTryllVoiceInput::HandleError (
    const FTryllError & Error
) 
```




<hr>



### function HandleTranscriptUpdate 

```C++
void FTryllVoiceInput::HandleTranscriptUpdate (
    const FString & Text,
    ETryllTranscriptUpdateKind Kind,
    int32 AudioMsConsumed
) 
```




<hr>



### function IsUtteranceActive 

```C++
inline bool FTryllVoiceInput::IsUtteranceActive () noexcept const
```




<hr>



### function IsValid 

```C++
inline bool FTryllVoiceInput::IsValid () noexcept const
```




<hr>



### function SendAudioBuffer [1/2]

```C++
void FTryllVoiceInput::SendAudioBuffer (
    const int16 * Samples,
    int32 NumSamples
) 
```



Push a block of mono int16 PCM to the server. Fire-and-forget; dropped silently when no utterance is active. 


        

<hr>



### function SendAudioBuffer [2/2]

```C++
inline void FTryllVoiceInput::SendAudioBuffer (
    const TArray< int16 > & Samples
) 
```




<hr>



### function SetOnError 

```C++
void FTryllVoiceInput::SetOnError (
    TFunction< void(const FTryllError &)> Callback
) 
```



Fired on protocol-level errors not tied to a Begin/End/Cancel callback. 


        

<hr>



### function SetOnTranscriptUpdate 

```C++
void FTryllVoiceInput::SetOnTranscriptUpdate (
    TFunction< void(const FTryllTranscriptUpdate &)> Callback
) 
```



Fired for each transcript update (partial and final). Invoked on the game thread. 


        

<hr>



### function ~FTryllVoiceInput 

```C++
FTryllVoiceInput::~FTryllVoiceInput () 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllVoiceInput.h`

