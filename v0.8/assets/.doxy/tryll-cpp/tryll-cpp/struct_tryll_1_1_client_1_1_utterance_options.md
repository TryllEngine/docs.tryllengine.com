

# Struct Tryll::Client::UtteranceOptions



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**UtteranceOptions**](struct_tryll_1_1_client_1_1_utterance_options.md)



_Per-utterance options passed to BeginUtterance._ 

* `#include <VoiceInput.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  bool | [**autoFinishOnSilence**](#variable-autofinishonsilence)   = `true`<br>_Let server VAD close the utterance._  |
|  std::uint32\_t | [**maxUtteranceMs**](#variable-maxutterancems)   = `60000`<br>_Hard timeout (ms) per utterance._  |












































## Public Attributes Documentation




### variable autoFinishOnSilence 

_Let server VAD close the utterance._ 
```C++
bool Tryll::Client::UtteranceOptions::autoFinishOnSilence;
```




<hr>



### variable maxUtteranceMs 

_Hard timeout (ms) per utterance._ 
```C++
std::uint32_t Tryll::Client::UtteranceOptions::maxUtteranceMs;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/cpp/include/tryll/VoiceInput.h`

