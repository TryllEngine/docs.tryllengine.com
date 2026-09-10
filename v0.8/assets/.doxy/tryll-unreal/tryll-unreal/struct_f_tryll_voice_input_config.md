

# Struct FTryllVoiceInputConfig



[**ClassList**](annotated.md) **>** [**FTryllVoiceInputConfig**](struct_f_tryll_voice_input_config.md)



[More...](#detailed-description)

* `#include <TryllVoiceInput.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  float | [**HotwordsScore**](#variable-hotwordsscore)   = `1.5f`<br> |
|  FString | [**HotwordsStoragePath**](#variable-hotwordsstoragepath)  <br> |
|  [**FTryllAudioFormat**](struct_f_tryll_audio_format.md) | [**InputFormat**](#variable-inputformat)  <br> |
|  FString | [**ModelName**](#variable-modelname)  <br> |
|  uint32 | [**VadMinSilenceMs**](#variable-vadminsilencems)   = `1000`<br> |
|  uint32 | [**VadSpeechPadMs**](#variable-vadspeechpadms)   = `250`<br> |
|  float | [**VadThreshold**](#variable-vadthreshold)   = `0.5f`<br> |












































## Detailed Description


Parameters for creating an [**FTryllVoiceInput**](class_f_tryll_voice_input.md) handle. 


    
## Public Attributes Documentation




### variable HotwordsScore 

```C++
float FTryllVoiceInputConfig::HotwordsScore;
```



How strongly matched phrases are weighted against one another during beam search (default 1.5). Measured on Parakeet TDT 0.6B v3: 1.8 recovered 5 of 8 target terms, 2.5 recovered 6 of 8. A lexicon line may override this for itself by ending with ` :<number>`. 


        

<hr>



### variable HotwordsStoragePath 

```C++
FString FTryllVoiceInputConfig::HotwordsStoragePath;
```



Relative path (resolved against the session storage folder) to a StringStorage (kind=List) config whose phrases the STT decoder biases toward. Loaded on demand. Empty (default) disables hotword biasing. 


        

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
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllVoiceInput.h`

