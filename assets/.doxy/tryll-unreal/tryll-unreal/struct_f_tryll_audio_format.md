

# Struct FTryllAudioFormat



[**ClassList**](annotated.md) **>** [**FTryllAudioFormat**](struct_f_tryll_audio_format.md)



[More...](#detailed-description)

* `#include <TryllVoiceInput.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  uint16 | [**BitsPerSample**](#variable-bitspersample)   = `16`<br> |
|  uint16 | [**Channels**](#variable-channels)   = `1`<br> |
|  uint32 | [**SampleRate**](#variable-samplerate)   = `16000`<br> |












































## Detailed Description


Audio format of the PCM samples pushed via SendAudioBuffer. The server resamples to the STT model's expected rate as needed. Plain C++ struct — not exposed to Blueprint. 


    
## Public Attributes Documentation




### variable BitsPerSample 

```C++
uint16 FTryllAudioFormat::BitsPerSample;
```




<hr>



### variable Channels 

```C++
uint16 FTryllAudioFormat::Channels;
```




<hr>



### variable SampleRate 

```C++
uint32 FTryllAudioFormat::SampleRate;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/TryllVoiceInput.h`

