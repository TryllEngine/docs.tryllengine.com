

# Struct Tryll::VoiceInputConfig



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**VoiceInputConfig**](struct_tryll_1_1_voice_input_config.md)



_Parameters for creating a_ [_**VoiceInput**_](class_tryll_1_1_voice_input.md) _handle._

* `#include <VoiceInput.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  float | [**hotwordsScore**](#variable-hotwordsscore)   = `1.5f`<br> |
|  std::string | [**hotwordsStorageName**](#variable-hotwordsstoragename)  <br> |
|  [**AudioFormat**](struct_tryll_1_1_audio_format.md) | [**inputFormat**](#variable-inputformat)  <br>_Format of samples passed to SendAudioBuffer._  |
|  std::string | [**modelName**](#variable-modelname)  <br>_Catalog name, e.g. "Whisper Tiny EN (int8)"._  |
|  std::uint32\_t | [**vadMinSilenceMs**](#variable-vadminsilencems)   = `500`<br>_Silence duration (ms) that closes a segment._  |
|  std::uint32\_t | [**vadSpeechPadMs**](#variable-vadspeechpadms)   = `250`<br> |
|  float | [**vadThreshold**](#variable-vadthreshold)   = `0.5f`<br>_Silero VAD speech-probability threshold._  |












































## Public Attributes Documentation




### variable hotwordsScore 

```C++
float Tryll::VoiceInputConfig::hotwordsScore;
```



Bonus score added to each matched hotword token during decoding. Typical range 1.0–2.5. Ignored when hotwordsStorageName is empty. 


        

<hr>



### variable hotwordsStorageName 

```C++
std::string Tryll::VoiceInputConfig::hotwordsStorageName;
```



Name of a session-scoped StringStorage (kind=List) whose entries are hotword phrases that the recognizer should be biased toward. Empty = no hotword biasing. Caller must register the storage via CreateStringStorage(...) before referencing it here. 


        

<hr>



### variable inputFormat 

_Format of samples passed to SendAudioBuffer._ 
```C++
AudioFormat Tryll::VoiceInputConfig::inputFormat;
```




<hr>



### variable modelName 

_Catalog name, e.g. "Whisper Tiny EN (int8)"._ 
```C++
std::string Tryll::VoiceInputConfig::modelName;
```




<hr>



### variable vadMinSilenceMs 

_Silence duration (ms) that closes a segment._ 
```C++
std::uint32_t Tryll::VoiceInputConfig::vadMinSilenceMs;
```




<hr>



### variable vadSpeechPadMs 

```C++
std::uint32_t Tryll::VoiceInputConfig::vadSpeechPadMs;
```



Padding (ms) added around detected speech. 


        

<hr>



### variable vadThreshold 

_Silero VAD speech-probability threshold._ 
```C++
float Tryll::VoiceInputConfig::vadThreshold;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-cpp/include/tryll/VoiceInput.h`

