

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
















## Public Functions

| Type | Name |
| ---: | :--- |
|  void | [**HandleAudio**](#function-handleaudio) (TArrayView&lt; const int16 &gt; Pcm) <br> |
|  void | [**HandleFormat**](#function-handleformat) (uint32 SampleRate, uint16 Channels, uint16 BitsPerSample) <br> |
|   | [**UTryllSpeakerComponent**](#function-utryllspeakercomponent) () <br> |
























## Protected Functions

| Type | Name |
| ---: | :--- |
| virtual void | [**BeginPlay**](#function-beginplay) () override<br> |
| virtual void | [**EndPlay**](#function-endplay) (const EEndPlayReason::Type EndPlayReason) override<br> |




## Detailed Description


## Public Attributes Documentation




### variable OnTtsAudioFinished 

```C++
FOnTryllTtsAudioFinished UTryllSpeakerComponent::OnTtsAudioFinished;
```



Fired when the underlying audio component reports playback completion. 


        

<hr>



### variable OnTtsAudioStarted 

```C++
FOnTryllTtsAudioStarted UTryllSpeakerComponent::OnTtsAudioStarted;
```



Fired once when the first PCM chunk of a turn is queued for playback. 


        

<hr>
## Public Functions Documentation




### function HandleAudio 

```C++
void UTryllSpeakerComponent::HandleAudio (
    TArrayView< const int16 > Pcm
) 
```



Called by [**UTryllAgentComponent**](class_u_tryll_agent_component.md) for each PCM chunk in the current turn. The view references caller-owned memory and is consumed synchronously. 


        

<hr>



### function HandleFormat 

```C++
void UTryllSpeakerComponent::HandleFormat (
    uint32 SampleRate,
    uint16 Channels,
    uint16 BitsPerSample
) 
```



Called by [**UTryllAgentComponent**](class_u_tryll_agent_component.md) when the server announces the audio format for the current turn. (Re)creates the procedural sound if the sample rate differs from the previous turn's. 


        

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

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllSpeakerComponent.h`

