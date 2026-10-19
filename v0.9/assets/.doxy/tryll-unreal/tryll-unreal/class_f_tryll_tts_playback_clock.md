

# Class FTryllTtsPlaybackClock



[**ClassList**](annotated.md) **>** [**FTryllTtsPlaybackClock**](class_f_tryll_tts_playback_clock.md)



[More...](#detailed-description)

* `#include <TryllSpeakerComponent.h>`





































## Public Functions

| Type | Name |
| ---: | :--- |
|  int64 | [**GetPlayedSamples**](#function-getplayedsamples) () const<br> |
|  int64 | [**GetQueuedSamples**](#function-getqueuedsamples) () const<br> |
|  void | [**OnSamplesQueued**](#function-onsamplesqueued) (int64 NumSamples) <br> |
|  void | [**SetWave**](#function-setwave) (USoundWaveProcedural \* InWave) <br> |




























## Detailed Description


Tracks how many TTS samples the audio device has actually consumed, so consumers pacing work against playback (e.g. lip sync) can slave to it. Safe to query from any thread; the speaker updates it on the game thread. Samples queued to a procedural sound that was discarded before draining (format change, EndPlay) count as played. 


    
## Public Functions Documentation




### function GetPlayedSamples 

```C++
int64 FTryllTtsPlaybackClock::GetPlayedSamples () const
```




<hr>



### function GetQueuedSamples 

```C++
int64 FTryllTtsPlaybackClock::GetQueuedSamples () const
```



Every sample ever queued, played or not. The next queued sample lands at this position. 


        

<hr>



### function OnSamplesQueued 

```C++
void FTryllTtsPlaybackClock::OnSamplesQueued (
    int64 NumSamples
) 
```




<hr>



### function SetWave 

```C++
void FTryllTtsPlaybackClock::SetWave (
    USoundWaveProcedural * InWave
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllSpeakerComponent.h`

