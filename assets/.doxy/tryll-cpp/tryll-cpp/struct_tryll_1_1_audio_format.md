

# Struct Tryll::AudioFormat



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**AudioFormat**](struct_tryll_1_1_audio_format.md)



[More...](#detailed-description)

* `#include <VoiceInput.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::uint16\_t | [**bitsPerSample**](#variable-bitspersample)   = `16`<br>_Only 16-bit PCM is supported._  |
|  std::uint16\_t | [**channels**](#variable-channels)   = `1`<br>_Channel count (mic capture is typically 1)._  |
|  std::uint32\_t | [**sampleRate**](#variable-samplerate)   = `16000`<br>_Sample rate in Hz (e.g. 48000 for mic capture)._  |












































## Detailed Description


Declares the audio format of samples the client will push via SendAudioBuffer. The server resamples to 16 kHz mono int16 as needed. 


    
## Public Attributes Documentation




### variable bitsPerSample 

_Only 16-bit PCM is supported._ 
```C++
std::uint16_t Tryll::AudioFormat::bitsPerSample;
```




<hr>



### variable channels 

_Channel count (mic capture is typically 1)._ 
```C++
std::uint16_t Tryll::AudioFormat::channels;
```




<hr>



### variable sampleRate 

_Sample rate in Hz (e.g. 48000 for mic capture)._ 
```C++
std::uint32_t Tryll::AudioFormat::sampleRate;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-cpp/include/tryll/VoiceInput.h`

