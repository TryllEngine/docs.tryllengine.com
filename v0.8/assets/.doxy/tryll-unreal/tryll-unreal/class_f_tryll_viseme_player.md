

# Class FTryllVisemePlayer



[**ClassList**](annotated.md) **>** [**FTryllVisemePlayer**](class_f_tryll_viseme_player.md)



[More...](#detailed-description)

* `#include <TryllVisemePlayer.h>`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FSettings**](struct_f_tryll_viseme_player_1_1_f_settings.md) <br> |






## Public Attributes

| Type | Name |
| ---: | :--- |
|  [**FSettings**](struct_f_tryll_viseme_player_1_1_f_settings.md) | [**Settings**](#variable-settings)  <br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  void | [**AddEvents**](#function-addevents) (TArrayView&lt; const [**FTryllVisemeEvent**](struct_f_tryll_viseme_event.md) &gt; Events) <br> |
|  void | [**ApplySettings**](#function-applysettings) (const [**FTryllVisemeSettings**](struct_f_tryll_viseme_settings.md) & In) <br> |
|  void | [**BeginTurn**](#function-beginturn) (int32 SampleRate, int64 TurnOriginSamples) <br> |
|   | [**FTryllVisemePlayer**](#function-ftryllvisemeplayer) () <br> |
|  ETryllViseme | [**GetActiveViseme**](#function-getactiveviseme) () const<br> |
|  const TArray&lt; FName &gt; & | [**GetChannels**](#function-getchannels) () const<br> |
|  float | [**GetCurve**](#function-getcurve) (FName Channel) const<br> |
|  TConstArrayView&lt; float &gt; | [**GetCurves**](#function-getcurves) () const<br> |
|  float | [**GetSmoothingLagMs**](#function-getsmoothinglagms) () const<br> |
|  void | [**Reset**](#function-reset) () <br> |
|  void | [**SetPoses**](#function-setposes) (TConstArrayView&lt; [**FTryllVisemePose**](struct_f_tryll_viseme_pose.md) &gt; Poses) <br> |
|  void | [**Tick**](#function-tick) (int64 PlayedSamples, float DeltaSeconds) <br> |


## Public Static Functions

| Type | Name |
| ---: | :--- |
|  TArray&lt; [**FTryllVisemePose**](struct_f_tryll_viseme_pose.md) &gt; | [**DefaultPoses**](#function-defaultposes) () <br> |


























## Detailed Description


Plays a TTS viseme track as ARKit curve values, in step with audio playback.


Engine-object free so any consumer can drive it: [**UTryllLipSyncComponent**](class_u_tryll_lip_sync_component.md) writes the curves to morph targets, and a MetaHuman integration can push the same curves through Epic's ARKit mapping.


Per turn: BeginTurn (from [**UTryllSpeakerComponent::OnTtsTurnStarted**](class_u_tryll_speaker_component.md#variable-onttsturnstarted)), then AddEvents for each viseme frame (OnTtsVisemes). Every frame: Tick with the playback clock's played samples, then read [**GetCurves()**](class_f_tryll_viseme_player.md#function-getcurves).


What it does, as in the Unity client's TryllLipSync:
* the active viseme's pose, scaled by the event weight (loudness);
* a smoothstep cross-fade over AttackDecayMs into each new viseme;
* Intensity scales everything;
* exponential smoothing at SmoothingPerSecond, ClosureSmoothingPerSecond while p/b/m or f/v is showing (a ~70 ms closure never lands at the slow rate);
* the track is read ahead by the delay that smoothing adds, so tuning it does not move sync;
* rest once the last shape of a turn has been held MaxHoldMs. 




    
## Public Attributes Documentation




### variable Settings 

```C++
FSettings FTryllVisemePlayer::Settings;
```




<hr>
## Public Functions Documentation




### function AddEvents 

```C++
void FTryllVisemePlayer::AddEvents (
    TArrayView< const FTryllVisemeEvent > Events
) 
```




<hr>



### function ApplySettings 

```C++
void FTryllVisemePlayer::ApplySettings (
    const FTryllVisemeSettings & In
) 
```



Copies the editable settings (a component's Details panel) into Settings. 


        

<hr>



### function BeginTurn 

```C++
void FTryllVisemePlayer::BeginTurn (
    int32 SampleRate,
    int64 TurnOriginSamples
) 
```




<hr>



### function FTryllVisemePlayer 

```C++
FTryllVisemePlayer::FTryllVisemePlayer () 
```




<hr>



### function GetActiveViseme 

```C++
inline ETryllViseme FTryllVisemePlayer::GetActiveViseme () const
```




<hr>



### function GetChannels 

```C++
inline const TArray< FName > & FTryllVisemePlayer::GetChannels () const
```



Every channel any pose drives; a consumer binds exactly these. 


        

<hr>



### function GetCurve 

```C++
float FTryllVisemePlayer::GetCurve (
    FName Channel
) const
```



Current value of one channel (0 if it isn't driven). 


        

<hr>



### function GetCurves 

```C++
inline TConstArrayView< float > FTryllVisemePlayer::GetCurves () const
```



Smoothed value per channel after the last Tick, 0..1, aligned with [**GetChannels()**](class_f_tryll_viseme_player.md#function-getchannels). 


        

<hr>



### function GetSmoothingLagMs 

```C++
float FTryllVisemePlayer::GetSmoothingLagMs () const
```



Look-ahead applied for the smoothing delay, in milliseconds. 


        

<hr>



### function Reset 

```C++
void FTryllVisemePlayer::Reset () 
```



Drops the track and eases back to rest. 


        

<hr>



### function SetPoses 

```C++
void FTryllVisemePlayer::SetPoses (
    TConstArrayView< FTryllVisemePose > Poses
) 
```



Replaces the pose table. Empty uses [**DefaultPoses()**](class_f_tryll_viseme_player.md#function-defaultposes). 


        

<hr>



### function Tick 

```C++
void FTryllVisemePlayer::Tick (
    int64 PlayedSamples,
    float DeltaSeconds
) 
```



Advances to the audible position. PlayedSamples is FTryllTtsPlaybackClock::GetPlayedSamples(). 


        

<hr>
## Public Static Functions Documentation




### function DefaultPoses 

```C++
static TArray< FTryllVisemePose > FTryllVisemePlayer::DefaultPoses () 
```



Hand-authored from published Oculus / VRChat viseme references (same table as the Unity client). 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllVisemePlayer.h`

