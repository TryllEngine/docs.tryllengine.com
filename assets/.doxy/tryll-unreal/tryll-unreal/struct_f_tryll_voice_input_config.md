

# Struct FTryllVoiceInputConfig



[**ClassList**](annotated.md) **>** [**FTryllVoiceInputConfig**](struct_f_tryll_voice_input_config.md)



[More...](#detailed-description)

* `#include <TryllVoiceInput.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  float | [**HotwordsScore**](#variable-hotwordsscore)   = `1.5f`<br> |
|  FString | [**HotwordsStorageName**](#variable-hotwordsstoragename)  <br> |
|  [**FTryllAudioFormat**](struct_f_tryll_audio_format.md) | [**InputFormat**](#variable-inputformat)  <br> |
|  FString | [**ModelName**](#variable-modelname)  <br> |
|  uint32 | [**VadMinSilenceMs**](#variable-vadminsilencems)   = `500`<br> |
|  uint32 | [**VadSpeechPadMs**](#variable-vadspeechpadms)   = `250`<br> |
|  float | [**VadThreshold**](#variable-vadthreshold)   = `0.5f`<br> |












































## Detailed Description


Parameters for creating an [**FTryllVoiceInput**](class_f_tryll_voice_input.md) handle. 


    
## Public Attributes Documentation




### variable HotwordsScore 

```C++
float FTryllVoiceInputConfig::HotwordsScore;
```



Per-token bias strength applied to every phrase (default 1.5). 1.5 = gentle; 2.5 = aggressive. 


        

<hr>



### variable HotwordsStorageName 

```C++
FString FTryllVoiceInputConfig::HotwordsStorageName;
```



Name of a StringStorage (kind=List) whose phrases the STT decoder biases toward. Empty (default) disables hotword biasing. 


        

<hr>



### variable InputFormat 

```C++
FTryllAudioFormat FTryllVoiceInputConfig::InputFormat;
```




<hr>



### variable ModelName 

```C++
FString FTryllVoiceInputConfig::ModelName;
```




<hr>



### variable VadMinSilenceMs 

```C++
uint32 FTryllVoiceInputConfig::VadMinSilenceMs;
```




<hr>



### variable VadSpeechPadMs 

```C++
uint32 FTryllVoiceInputConfig::VadSpeechPadMs;
```




<hr>



### variable VadThreshold 

```C++
float FTryllVoiceInputConfig::VadThreshold;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/TryllVoiceInput.h`

