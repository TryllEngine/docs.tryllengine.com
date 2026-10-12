

# Struct FTryllVisemeEvent



[**ClassList**](annotated.md) **>** [**FTryllVisemeEvent**](struct_f_tryll_viseme_event.md)



[More...](#detailed-description)

* `#include <TryllVisemeEvent.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  int64 | [**SampleOffset**](#variable-sampleoffset)   = `0`<br> |
|  ETryllViseme | [**Viseme**](#variable-viseme)   = `ETryllViseme::sil`<br> |
|  float | [**Weight**](#variable-weight)   = `1.0f`<br> |












































## Detailed Description


One step of a TTS viseme track. The viseme holds until the next event's offset (or the end of its sentence). 


    
## Public Attributes Documentation




### variable SampleOffset 

```C++
int64 FTryllVisemeEvent::SampleOffset;
```



Samples from the start of the turn's TTS audio, at the rate announced by the audio format. 


        

<hr>



### variable Viseme 

```C++
ETryllViseme FTryllVisemeEvent::Viseme;
```




<hr>



### variable Weight 

```C++
float FTryllVisemeEvent::Weight;
```



How far the shape opens (0..1), from the loudness of that stretch of speech. Closures and rests are always 1. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllVisemeEvent.h`

