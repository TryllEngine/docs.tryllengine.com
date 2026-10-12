

# Struct FTryllUtteranceOptions



[**ClassList**](annotated.md) **>** [**FTryllUtteranceOptions**](struct_f_tryll_utterance_options.md)



[More...](#detailed-description)

* `#include <TryllVoiceInput.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  uint32 | [**MaxUtteranceMs**](#variable-maxutterancems)   = `60000`<br> |
|  bool | [**bAutoFinishOnSilence**](#variable-bautofinishonsilence)   = `true`<br> |












































## Detailed Description


Per-utterance options passed to BeginUtterance. 


    
## Public Attributes Documentation




### variable MaxUtteranceMs 

```C++
uint32 FTryllUtteranceOptions::MaxUtteranceMs;
```



Hard timeout (ms) per utterance. 


        

<hr>



### variable bAutoFinishOnSilence 

```C++
bool FTryllUtteranceOptions::bAutoFinishOnSilence;
```



Let server VAD close the utterance on silence. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllVoiceInput.h`

