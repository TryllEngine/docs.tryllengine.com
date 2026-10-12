

# Class FTryllTtsPlaybackEvents



[**ClassList**](annotated.md) **>** [**FTryllTtsPlaybackEvents**](class_f_tryll_tts_playback_events.md)










































## Public Functions

| Type | Name |
| ---: | :--- |
|  bool | [**IsAwaitingDrain**](#function-isawaitingdrain) () const<br> |
|  bool | [**IsSuppressed**](#function-issuppressed) () const<br> |
|  bool | [**OnAudioQueued**](#function-onaudioqueued) () <br> |
|  void | [**OnFormat**](#function-onformat) () <br> |
|  bool | [**OnQueueLevel**](#function-onqueuelevel) (int64 QueuedBytes) <br> |
|  bool | [**OnStop**](#function-onstop) () <br> |
|  void | [**OnTurnComplete**](#function-onturncomplete) () <br> |
|  void | [**Reset**](#function-reset) () <br> |




























## Public Functions Documentation




### function IsAwaitingDrain 

```C++
inline bool FTryllTtsPlaybackEvents::IsAwaitingDrain () const
```




<hr>



### function IsSuppressed 

```C++
inline bool FTryllTtsPlaybackEvents::IsSuppressed () const
```




<hr>



### function OnAudioQueued 

```C++
bool FTryllTtsPlaybackEvents::OnAudioQueued () 
```



Returns true when OnTtsAudioStarted should fire. 


        

<hr>



### function OnFormat 

```C++
void FTryllTtsPlaybackEvents::OnFormat () 
```




<hr>



### function OnQueueLevel 

```C++
bool FTryllTtsPlaybackEvents::OnQueueLevel (
    int64 QueuedBytes
) 
```



Returns true when OnTtsAudioFinished should fire. 


        

<hr>



### function OnStop 

```C++
bool FTryllTtsPlaybackEvents::OnStop () 
```



Returns true when OnTtsAudioFinished should fire. 


        

<hr>



### function OnTurnComplete 

```C++
void FTryllTtsPlaybackEvents::OnTurnComplete () 
```




<hr>



### function Reset 

```C++
void FTryllTtsPlaybackEvents::Reset () 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllTtsPlaybackEvents.h`

