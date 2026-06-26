

# Struct Tryll::Client::VoiceInputConfig



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**VoiceInputConfig**](struct_tryll_1_1_client_1_1_voice_input_config.md)



_Parameters for creating a_ [_**VoiceInput**_](class_tryll_1_1_client_1_1_voice_input.md) _handle._

* `#include <VoiceInput.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  float | [**hotwordsScore**](#variable-hotwordsscore)   = `1.5f`<br> |
|  std::string | [**hotwordsStoragePath**](#variable-hotwordsstoragepath)  <br> |
|  [**AudioFormat**](struct_tryll_1_1_client_1_1_audio_format.md) | [**inputFormat**](#variable-inputformat)  <br>_Format of samples passed to SendAudioBuffer._  |
|  std::string | [**modelName**](#variable-modelname)  <br>_Catalog name, e.g. "Whisper Tiny EN (int8)"._  |
|  std::uint32\_t | [**vadMinSilenceMs**](#variable-vadminsilencems)   = `500`<br>_Silence duration (ms) that closes a segment._  |
|  std::uint32\_t | [**vadSpeechPadMs**](#variable-vadspeechpadms)   = `250`<br> |
|  float | [**vadThreshold**](#variable-vadthreshold)   = `0.5f`<br>_Silero VAD speech-probability threshold._  |












































## Public Attributes Documentation




### variable hotwordsScore 

```C++
float Tryll::Client::VoiceInputConfig::hotwordsScore;
```



Bonus score added to each matched hotword token during decoding. Typical range 1.0–2.5. Ignored when hotwordsStoragePath is empty. 


        

<hr>



### variable hotwordsStoragePath 

```C++
std::string Tryll::Client::VoiceInputConfig::hotwordsStoragePath;
```



Relative path (resolved against the session storage folder) to a List StringStorage (.txt) whose entries are hotword phrases that the recognizer should be biased toward. Empty = no hotword biasing. The file loads on demand server-side — no prior CreateStringStorage needed. 


        

<hr>



### variable inputFormat 

_Format of samples passed to SendAudioBuffer._ 
```C++
AudioFormat Tryll::Client::VoiceInputConfig::inputFormat;
```




<hr>



### variable modelName 

_Catalog name, e.g. "Whisper Tiny EN (int8)"._ 
```C++
std::string Tryll::Client::VoiceInputConfig::modelName;
```




<hr>



### variable vadMinSilenceMs 

_Silence duration (ms) that closes a segment._ 
```C++
std::uint32_t Tryll::Client::VoiceInputConfig::vadMinSilenceMs;
```




<hr>



### variable vadSpeechPadMs 

```C++
std::uint32_t Tryll::Client::VoiceInputConfig::vadSpeechPadMs;
```



Padding (ms) added around detected speech. 


        

<hr>



### variable vadThreshold 

_Silero VAD speech-probability threshold._ 
```C++
float Tryll::Client::VoiceInputConfig::vadThreshold;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/VoiceInput.h`

