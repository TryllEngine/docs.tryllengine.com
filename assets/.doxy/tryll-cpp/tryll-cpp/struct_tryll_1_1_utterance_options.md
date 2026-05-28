

# Struct Tryll::UtteranceOptions



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**UtteranceOptions**](struct_tryll_1_1_utterance_options.md)



_Per-utterance options passed to BeginUtterance._ 

* `#include <VoiceInput.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  bool | [**autoFinishOnSilence**](#variable-autofinishonsilence)   = `true`<br>_Let server VAD close the utterance._  |
|  [**AgentProxy**](class_tryll_1_1_agent_proxy.md) \* | [**autoSendTarget**](#variable-autosendtarget)   = `nullptr`<br> |
|  std::uint32\_t | [**maxUtteranceMs**](#variable-maxutterancems)   = `60000`<br>_Hard timeout (ms) per utterance._  |












































## Public Attributes Documentation




### variable autoFinishOnSilence 

_Let server VAD close the utterance._ 
```C++
bool Tryll::UtteranceOptions::autoFinishOnSilence;
```




<hr>



### variable autoSendTarget 

```C++
AgentProxy* Tryll::UtteranceOptions::autoSendTarget;
```



When non-null the server forwards the final transcript to this agent. null = transcribe-only mode (no autosend). 


        

<hr>



### variable maxUtteranceMs 

_Hard timeout (ms) per utterance._ 
```C++
std::uint32_t Tryll::UtteranceOptions::maxUtteranceMs;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-cpp/include/tryll/VoiceInput.h`

