

# File TryllSpeakerComponent.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllSpeakerComponent.h**](_tryll_speaker_component_8h.md)





* `#include "CoreMinimal.h"`
* `#include "Components/ActorComponent.h"`
* `#include "TryllTtsPlaybackEvents.h"`
* `#include "TryllVisemeEvent.h"`
* `#include "TryllSpeakerComponent.generated.h"`















## Classes

| Type | Name |
| ---: | :--- |
| class | [**FTryllTtsPlaybackClock**](class_f_tryll_tts_playback_clock.md) <br> |
| class | [**UTryllSpeakerComponent**](class_u_tryll_speaker_component.md) <br> |






















## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_multicast_delegate_twoparams) (FOnTryllTtsPcm, const TArray&lt; float &gt; &, int32) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_multicast_delegate_twoparams) (FOnTryllTtsTurnStarted, int32, int64) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_multicast_delegate_twoparams) (FOnTryllTtsVisemes, const FString &, TArrayView&lt; const [**FTryllVisemeEvent**](struct_f_tryll_viseme_event.md) &gt;) <br> |




























## Public Functions Documentation




### function DECLARE\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_MULTICAST_DELEGATE_TwoParams (
    FOnTryllTtsPcm,
    const TArray< float > &,
    int32
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_MULTICAST_DELEGATE_TwoParams (
    FOnTryllTtsTurnStarted,
    int32,
    int64
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_MULTICAST_DELEGATE_TwoParams (
    FOnTryllTtsVisemes,
    const FString &,
    TArrayView< const FTryllVisemeEvent >
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllSpeakerComponent.h`

