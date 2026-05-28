

# Class Tryll::VoiceInput



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**VoiceInput**](class_tryll_1_1_voice_input.md)



[More...](#detailed-description)

* `#include <VoiceInput.h>`

















## Public Types

| Type | Name |
| ---: | :--- |
| typedef std::function&lt; void(const [**TranscriptUpdate**](struct_tryll_1_1_transcript_update.md) &)&gt; | [**TranscriptCallback**](#typedef-transcriptcallback)  <br> |




















## Public Functions

| Type | Name |
| ---: | :--- |
|  void | [**BeginUtterance**](#function-beginutterance) ([**UtteranceOptions**](struct_tryll_1_1_utterance_options.md) opts={}, std::chrono::milliseconds timeout=std::chrono::seconds{10}) <br> |
|  std::future&lt; void &gt; | [**BeginUtteranceAsync**](#function-beginutteranceasync) ([**UtteranceOptions**](struct_tryll_1_1_utterance_options.md) opts={}) <br>_Open a new utterance. Rejects if one is already active (throws_ [_**TryllError**_](class_tryll_1_1_tryll_error.md) _)._ |
|  void | [**CancelUtterance**](#function-cancelutterance) (std::chrono::milliseconds timeout=std::chrono::seconds{10}) <br> |
|  std::future&lt; void &gt; | [**CancelUtteranceAsync**](#function-cancelutteranceasync) () <br>_Cancel the utterance without producing a transcript._  |
|  void | [**EndUtterance**](#function-endutterance) (std::chrono::milliseconds timeout=std::chrono::seconds{60}) <br> |
|  std::future&lt; void &gt; | [**EndUtteranceAsync**](#function-endutteranceasync) () <br> |
|  std::uint64\_t | [**GetVoiceInputId**](#function-getvoiceinputid) () noexcept const<br>_The server-assigned handle id (zero before the handle is valid)._  |
|  bool | [**IsUtteranceActive**](#function-isutteranceactive) () noexcept const<br>_True if BeginUtterance has been called and the utterance is not yet complete._  |
|  void | [**SendAudioBuffer**](#function-sendaudiobuffer) (std::span&lt; const std::int16\_t &gt; pcm) <br> |
|  void | [**SetOnTranscriptUpdate**](#function-setontranscriptupdate) (TranscriptCallback cb) <br> |
|   | [**VoiceInput**](#function-voiceinput-13) (const VoiceInput &) = delete<br> |
|   | [**VoiceInput**](#function-voiceinput-23) (VoiceInput &&) noexcept<br> |
|  VoiceInput & | [**operator=**](#function-operator) (const VoiceInput &) = delete<br> |
|  VoiceInput & | [**operator=**](#function-operator_1) (VoiceInput &&) noexcept<br> |
|   | [**~VoiceInput**](#function-voiceinput) () <br>_Implicitly cancels any active utterance and destroys the server-side handle._  |




























## Detailed Description


Move-only handle to a server-side [**VoiceInput**](class_tryll_1_1_voice_input.md) session.


Obtained via [**TryllClient::CreateVoiceInput**](class_tryll_1_1_tryll_client.md#function-createvoiceinput) or CreateVoiceInputAsync.


Lifetime guarantee: the destructor issues an implicit CancelUtterance (if active) followed by DestroyVoiceInput so the server-side ISttSession is always cleaned up. Move the handle to extend its lifetime.


Thread-safety: this handle is NOT internally synchronized. All public methods — including SendAudioBuffer — must be called from the same thread, or the caller must provide external synchronization. Moving or destroying the handle must likewise be serialized with any concurrent method calls. 


    
## Public Types Documentation




### typedef TranscriptCallback 

```C++
using Tryll::VoiceInput::TranscriptCallback = std::function<void(const TranscriptUpdate&)>;
```




<hr>
## Public Functions Documentation




### function BeginUtterance 

```C++
void Tryll::VoiceInput::BeginUtterance (
    UtteranceOptions opts={},
    std::chrono::milliseconds timeout=std::chrono::seconds{10}
) 
```




<hr>



### function BeginUtteranceAsync 

_Open a new utterance. Rejects if one is already active (throws_ [_**TryllError**_](class_tryll_1_1_tryll_error.md) _)._
```C++
std::future< void > Tryll::VoiceInput::BeginUtteranceAsync (
    UtteranceOptions opts={}
) 
```




<hr>



### function CancelUtterance 

```C++
void Tryll::VoiceInput::CancelUtterance (
    std::chrono::milliseconds timeout=std::chrono::seconds{10}
) 
```




<hr>



### function CancelUtteranceAsync 

_Cancel the utterance without producing a transcript._ 
```C++
std::future< void > Tryll::VoiceInput::CancelUtteranceAsync () 
```




<hr>



### function EndUtterance 

```C++
void Tryll::VoiceInput::EndUtterance (
    std::chrono::milliseconds timeout=std::chrono::seconds{60}
) 
```




<hr>



### function EndUtteranceAsync 

```C++
std::future< void > Tryll::VoiceInput::EndUtteranceAsync () 
```



Commit the utterance (push-to-talk release). The engine drains pending audio, decodes, and delivers the final [**TranscriptUpdate**](struct_tryll_1_1_transcript_update.md). 


        

<hr>



### function GetVoiceInputId 

_The server-assigned handle id (zero before the handle is valid)._ 
```C++
std::uint64_t Tryll::VoiceInput::GetVoiceInputId () noexcept const
```




<hr>



### function IsUtteranceActive 

_True if BeginUtterance has been called and the utterance is not yet complete._ 
```C++
bool Tryll::VoiceInput::IsUtteranceActive () noexcept const
```




<hr>



### function SendAudioBuffer 

```C++
void Tryll::VoiceInput::SendAudioBuffer (
    std::span< const std::int16_t > pcm
) 
```



Push a block of mono int16 PCM samples to the server. Fire-and-forget (no ack expected). Thread-safe. 


        

<hr>



### function SetOnTranscriptUpdate 

```C++
void Tryll::VoiceInput::SetOnTranscriptUpdate (
    TranscriptCallback cb
) 
```



Register a callback for transcript updates. Called on the reader thread — keep brief and non-blocking. Pass nullptr to clear. 


        

<hr>



### function VoiceInput [1/3]

```C++
Tryll::VoiceInput::VoiceInput (
    const VoiceInput &
) = delete
```




<hr>



### function VoiceInput [2/3]

```C++
Tryll::VoiceInput::VoiceInput (
    VoiceInput &&
) noexcept
```




<hr>



### function operator= 

```C++
VoiceInput & Tryll::VoiceInput::operator= (
    const VoiceInput &
) = delete
```




<hr>



### function operator= 

```C++
VoiceInput & Tryll::VoiceInput::operator= (
    VoiceInput &&
) noexcept
```




<hr>



### function ~VoiceInput 

_Implicitly cancels any active utterance and destroys the server-side handle._ 
```C++
Tryll::VoiceInput::~VoiceInput () 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-cpp/include/tryll/VoiceInput.h`

