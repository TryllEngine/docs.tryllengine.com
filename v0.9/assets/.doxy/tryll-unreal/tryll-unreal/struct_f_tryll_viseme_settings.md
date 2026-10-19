

# Struct FTryllVisemeSettings



[**ClassList**](annotated.md) **>** [**FTryllVisemeSettings**](struct_f_tryll_viseme_settings.md)



[More...](#detailed-description)

* `#include <TryllVisemeEvent.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  float | [**AttackDecayMs**](#variable-attackdecayms)   = `50.0f`<br> |
|  float | [**ClosureSmoothingPerSecond**](#variable-closuresmoothingpersecond)   = `60.0f`<br> |
|  float | [**Intensity**](#variable-intensity)   = `0.4f`<br> |
|  float | [**MaxHoldMs**](#variable-maxholdms)   = `400.0f`<br> |
|  float | [**SmoothingPerSecond**](#variable-smoothingpersecond)   = `16.0f`<br> |
|  float | [**SyncOffsetMs**](#variable-syncoffsetms)   = `0.0f`<br> |
|  bool | [**bCompensateOutputLatency**](#variable-bcompensateoutputlatency)   = `true`<br> |
|  bool | [**bCompensateSmoothingLag**](#variable-bcompensatesmoothinglag)   = `true`<br> |












































## Detailed Description


How a viseme track is played on a face. Shared by every Tryll lip-sync component. 


    
## Public Attributes Documentation




### variable AttackDecayMs 

```C++
float FTryllVisemeSettings::AttackDecayMs;
```



Cross-fade between consecutive shapes, in milliseconds. 


        

<hr>



### variable ClosureSmoothingPerSecond 

```C++
float FTryllVisemeSettings::ClosureSmoothingPerSecond;
```



Faster rate used while a lip closure (p, b, m, f, v) is showing, so short closures actually land. 


        

<hr>



### variable Intensity 

```C++
float FTryllVisemeSettings::Intensity;
```



Scales every pose. 0 = no movement, 1 = the full pose table. 


        

<hr>



### variable MaxHoldMs 

```C++
float FTryllVisemeSettings::MaxHoldMs;
```



Returns to rest if a turn ends without its closing rest (a cancelled turn). 0 disables. 


        

<hr>



### variable SmoothingPerSecond 

```C++
float FTryllVisemeSettings::SmoothingPerSecond;
```



How quickly the face moves toward each shape. Lower is softer, higher is snappier. 0 disables smoothing. 


        

<hr>



### variable SyncOffsetMs 

```C++
float FTryllVisemeSettings::SyncOffsetMs;
```



Shifts the face against the audio. Positive moves the face earlier. 


        

<hr>



### variable bCompensateOutputLatency 

```C++
bool FTryllVisemeSettings::bCompensateOutputLatency;
```



Holds the face back by the audio engine's output buffer, so it lines up with the sound as heard. 


        

<hr>



### variable bCompensateSmoothingLag 

```C++
bool FTryllVisemeSettings::bCompensateSmoothingLag;
```



Reads the track ahead by the delay the smoothing adds, so tuning smoothing does not shift sync. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllVisemeEvent.h`

