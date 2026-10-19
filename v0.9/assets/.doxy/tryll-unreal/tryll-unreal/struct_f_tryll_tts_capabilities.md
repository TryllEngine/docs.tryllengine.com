

# Struct FTryllTtsCapabilities



[**ClassList**](annotated.md) **>** [**FTryllTtsCapabilities**](struct_f_tryll_tts_capabilities.md)



[More...](#detailed-description)

* `#include <TryllModelInfo.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  TArray&lt; [**FTryllTtsInlineTag**](struct_f_tryll_tts_inline_tag.md) &gt; | [**InlineTags**](#variable-inlinetags)  <br> |
|  int32 | [**NumSpeakers**](#variable-numspeakers)   = `0`<br> |
|  TArray&lt; [**FTryllTtsVoicePreset**](struct_f_tryll_tts_voice_preset.md) &gt; | [**VoicePresets**](#variable-voicepresets)  <br> |
|  bool | [**bDeliveryInstruction**](#variable-bdeliveryinstruction)   = `false`<br> |
|  bool | [**bExpressiveness**](#variable-bexpressiveness)   = `false`<br> |
|  bool | [**bLanguagePerCall**](#variable-blanguagepercall)   = `false`<br> |
|  bool | [**bReferenceVoice**](#variable-breferencevoice)   = `false`<br> |
|  bool | [**bSpeed**](#variable-bspeed)   = `false`<br> |
|  bool | [**bVoiceDescription**](#variable-bvoicedescription)   = `false`<br> |












































## Detailed Description


Structured TTS capabilities advertised by the catalog. 


    
## Public Attributes Documentation




### variable InlineTags 

```C++
TArray<FTryllTtsInlineTag> FTryllTtsCapabilities::InlineTags;
```




<hr>



### variable NumSpeakers 

```C++
int32 FTryllTtsCapabilities::NumSpeakers;
```



Number of speaker ids the model accepts (Speaker Id must be below it); 0 = not declared. 


        

<hr>



### variable VoicePresets 

```C++
TArray<FTryllTtsVoicePreset> FTryllTtsCapabilities::VoicePresets;
```




<hr>



### variable bDeliveryInstruction 

```C++
bool FTryllTtsCapabilities::bDeliveryInstruction;
```




<hr>



### variable bExpressiveness 

```C++
bool FTryllTtsCapabilities::bExpressiveness;
```



Chatterbox-style emotion scalar: the voice's Expressiveness and a delivery mode's Expressiveness apply. 


        

<hr>



### variable bLanguagePerCall 

```C++
bool FTryllTtsCapabilities::bLanguagePerCall;
```




<hr>



### variable bReferenceVoice 

```C++
bool FTryllTtsCapabilities::bReferenceVoice;
```




<hr>



### variable bSpeed 

```C++
bool FTryllTtsCapabilities::bSpeed;
```




<hr>



### variable bVoiceDescription 

```C++
bool FTryllTtsCapabilities::bVoiceDescription;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllModelInfo.h`

