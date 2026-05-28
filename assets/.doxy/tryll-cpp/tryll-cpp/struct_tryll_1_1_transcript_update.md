

# Struct Tryll::TranscriptUpdate



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**TranscriptUpdate**](struct_tryll_1_1_transcript_update.md)



_Transcript update delivered via the_ [_**VoiceInput**_](class_tryll_1_1_voice_input.md) _callback._

* `#include <VoiceInput.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::uint32\_t | [**audioMsConsumed**](#variable-audiomsconsumed)   = `0`<br>_Milliseconds of audio processed so far._  |
|  ::Tryll::TranscriptUpdateKind | [**kind**](#variable-kind)   = `::Tryll::TranscriptUpdateKind\_Partial`<br>_Event kind._  |
|  std::string | [**text**](#variable-text)  <br>_Recognized text; empty for SpeechStart._  |












































## Public Attributes Documentation




### variable audioMsConsumed 

_Milliseconds of audio processed so far._ 
```C++
std::uint32_t Tryll::TranscriptUpdate::audioMsConsumed;
```




<hr>



### variable kind 

_Event kind._ 
```C++
::Tryll::TranscriptUpdateKind Tryll::TranscriptUpdate::kind;
```




<hr>



### variable text 

_Recognized text; empty for SpeechStart._ 
```C++
std::string Tryll::TranscriptUpdate::text;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-cpp/include/tryll/VoiceInput.h`

