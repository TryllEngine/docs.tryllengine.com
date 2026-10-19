

# Class UTryllSpeakerComponent



[**ClassList**](annotated.md) **>** [**UTryllSpeakerComponent**](class_u_tryll_speaker_component.md)



[More...](#detailed-description)

* `#include <TryllSpeakerComponent.h>`



Inherits the following classes: UActorComponent


















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FOnTryllTtsAudioFinished | [**OnTtsAudioFinished**](#variable-onttsaudiofinished)  <br> |
|  FOnTryllTtsAudioStarted | [**OnTtsAudioStarted**](#variable-onttsaudiostarted)  <br> |
|  FOnTryllTtsPcm | [**OnTtsPcm**](#variable-onttspcm)  <br> |
|  FOnTryllTtsTurnStarted | [**OnTtsTurnStarted**](#variable-onttsturnstarted)  <br> |
|  FOnTryllTtsVisemes | [**OnTtsVisemes**](#variable-onttsvisemes)  <br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  float | [**GetOutputLatencyMs**](#function-getoutputlatencyms) () const<br> |
|  TSharedRef&lt; [**FTryllTtsPlaybackClock**](class_f_tryll_tts_playback_clock.md) &gt; | [**GetPlaybackClock**](#function-getplaybackclock) () const<br> |
|  int32 | [**GetSampleRate**](#function-getsamplerate) () const<br> |
|  void | [**HandleAudio**](#function-handleaudio) (const FString & NodeName, TArrayView&lt; const int16 &gt; Pcm) <br> |
|  void | [**HandleFormat**](#function-handleformat) (const FString & NodeName, uint32 SampleRate, uint16 Channels, uint16 BitsPerSample) <br> |
|  void | [**HandleTurnComplete**](#function-handleturncomplete) () <br> |
|  void | [**HandleVisemes**](#function-handlevisemes) (const FString & NodeName, TArrayView&lt; const [**FTryllVisemeEvent**](struct_f_tryll_viseme_event.md) &gt; Events) <br> |
|  void | [**StopPlayback**](#function-stopplayback) () <br> |
|   | [**UTryllSpeakerComponent**](#function-utryllspeakercomponent) () <br> |
























## Protected Functions

| Type | Name |
| ---: | :--- |
| virtual void | [**BeginPlay**](#function-beginplay) () override<br> |
| virtual void | [**EndPlay**](#function-endplay) (const EEndPlayReason::Type EndPlayReason) override<br> |
| virtual void | [**TickComponent**](#function-tickcomponent) (float DeltaTime, ELevelTick TickType, FActorComponentTickFunction \* ThisTickFunction) override<br> |




## Detailed Description


## Public Attributes Documentation




### variable OnTtsAudioFinished 

```C++
FOnTryllTtsAudioFinished UTryllSpeakerComponent::OnTtsAudioFinished;
```



Fired when the turn has completed and its queued audio has been handed to the audio engine — the engine's own buffers (a few tens of ms) are still to play out. Also fired immediately by StopPlayback while speaking. 


        

<hr>



### variable OnTtsAudioStarted 

```C++
FOnTryllTtsAudioStarted UTryllSpeakerComponent::OnTtsAudioStarted;
```



Fired once when the first PCM chunk of a turn is queued for playback. Always followed by exactly one OnTtsAudioFinished before it fires again. 


        

<hr>



### variable OnTtsPcm 

```C++
FOnTryllTtsPcm UTryllSpeakerComponent::OnTtsPcm;
```



Fired on the game thread for each PCM chunk after it is queued for playback, as normalized float samples in [-1, 1]. The array is a reused scratch buffer — copy it if you need it beyond the callback. 


        

<hr>



### variable OnTtsTurnStarted 

```C++
FOnTryllTtsTurnStarted UTryllSpeakerComponent::OnTtsTurnStarted;
```



Fired on the game thread when a turn's audio format is announced, before any of its audio is queued. TurnOriginSamples is the playback-clock position (GetPlaybackClock()-&gt;GetPlayedSamples() units) at which the turn's first sample will play; viseme offsets are relative to it. 


        

<hr>



### variable OnTtsVisemes 

```C++
FOnTryllTtsVisemes UTryllSpeakerComponent::OnTtsVisemes;
```



Fired on the game thread for each viseme frame. The view is valid only during the callback. 


        

<hr>
## Public Functions Documentation




### function GetOutputLatencyMs 

```C++
float UTryllSpeakerComponent::GetOutputLatencyMs () const
```



Audio engine output buffering after the playback clock's played count, in milliseconds. 0 without an audio device. 


        

<hr>



### function GetPlaybackClock 

```C++
inline TSharedRef< FTryllTtsPlaybackClock > UTryllSpeakerComponent::GetPlaybackClock () const
```




<hr>



### function GetSampleRate 

```C++
inline int32 UTryllSpeakerComponent::GetSampleRate () const
```




<hr>



### function HandleAudio 

```C++
void UTryllSpeakerComponent::HandleAudio (
    const FString & NodeName,
    TArrayView< const int16 > Pcm
) 
```



Called by [**UTryllAgentComponent**](class_u_tryll_agent_component.md) for each PCM chunk in the current turn. The view references caller-owned memory and is consumed synchronously. NodeName is accepted for signature compatibility (see HandleFormat) but unused. 


        

<hr>



### function HandleFormat 

```C++
void UTryllSpeakerComponent::HandleFormat (
    const FString & NodeName,
    uint32 SampleRate,
    uint16 Channels,
    uint16 BitsPerSample
) 
```



Called by [**UTryllAgentComponent**](class_u_tryll_agent_component.md) when the server announces the audio format for the current turn. (Re)creates the procedural sound if the sample rate differs from the previous turn's. NodeName (the producing node's name — multi-sender attribution) is accepted for signature compatibility with FTryllAgent::OnTtsAudioFormat but otherwise unused: this component mixes all producers into one procedural sound. 


        

<hr>



### function HandleTurnComplete 

```C++
void UTryllSpeakerComponent::HandleTurnComplete () 
```



Called by [**UTryllAgentComponent**](class_u_tryll_agent_component.md) when no more audio is coming for the current turn: on TurnComplete, and when the agent is destroyed or the connection drops. 


        

<hr>



### function HandleVisemes 

```C++
void UTryllSpeakerComponent::HandleVisemes (
    const FString & NodeName,
    TArrayView< const FTryllVisemeEvent > Events
) 
```



Called by [**UTryllAgentComponent**](class_u_tryll_agent_component.md) for each viseme frame (one per synthesized sentence, when the agent's voice profile has Emit Visemes on). Re-broadcast on OnTtsVisemes so lip sync needs only this speaker: it already owns the audio and the playback clock the track is timed against. 


        

<hr>



### function StopPlayback 

```C++
void UTryllSpeakerComponent::StopPlayback () 
```



Cancels the current playback: drops all queued audio and ignores further audio until the next turn announces its audio format. Fires OnTtsAudioFinished if speech was playing. Does not cancel the agent's turn — call Cancel on the agent for that. Safe to call repeatedly and when nothing is playing. 


        

<hr>



### function UTryllSpeakerComponent 

```C++
UTryllSpeakerComponent::UTryllSpeakerComponent () 
```




<hr>
## Protected Functions Documentation




### function BeginPlay 

```C++
virtual void UTryllSpeakerComponent::BeginPlay () override
```




<hr>



### function EndPlay 

```C++
virtual void UTryllSpeakerComponent::EndPlay (
    const EEndPlayReason::Type EndPlayReason
) override
```




<hr>



### function TickComponent 

```C++
virtual void UTryllSpeakerComponent::TickComponent (
    float DeltaTime,
    ELevelTick TickType,
    FActorComponentTickFunction * ThisTickFunction
) override
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllSpeakerComponent.h`

