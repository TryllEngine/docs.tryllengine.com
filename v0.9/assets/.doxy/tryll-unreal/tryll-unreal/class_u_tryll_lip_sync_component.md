

# Class UTryllLipSyncComponent



[**ClassList**](annotated.md) **>** [**UTryllLipSyncComponent**](class_u_tryll_lip_sync_component.md)



[More...](#detailed-description)

* `#include <TryllLipSyncComponent.h>`



Inherits the following classes: UActorComponent


















## Public Attributes

| Type | Name |
| ---: | :--- |
|  TObjectPtr&lt; USkeletalMeshComponent &gt; | [**FaceMesh**](#variable-facemesh)  <br> |
|  TArray&lt; [**FTryllVisemePose**](struct_f_tryll_viseme_pose.md) &gt; | [**Poses**](#variable-poses)  <br> |
|  [**FTryllVisemeSettings**](struct_f_tryll_viseme_settings.md) | [**Settings**](#variable-settings)  <br> |
|  TObjectPtr&lt; [**UTryllSpeakerComponent**](class_u_tryll_speaker_component.md) &gt; | [**Speaker**](#variable-speaker)  <br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  void | [**BindFace**](#function-bindface) (USkeletalMeshComponent \* InFaceMesh) <br> |
|  ETryllViseme | [**GetActiveViseme**](#function-getactiveviseme) () const<br> |
|  float | [**GetAppliedJawOpen**](#function-getappliedjawopen) () const<br> |
|  int32 | [**GetBoundChannelCount**](#function-getboundchannelcount) () const<br> |
|  const [**FTryllVisemePlayer**](class_f_tryll_viseme_player.md) & | [**GetPlayer**](#function-getplayer) () const<br> |
|  bool | [**IsFaceBound**](#function-isfacebound) () const<br> |
|  void | [**SetIntensity**](#function-setintensity) (float InIntensity) <br> |
|   | [**UTryllLipSyncComponent**](#function-utrylllipsynccomponent) () <br> |
























## Protected Functions

| Type | Name |
| ---: | :--- |
| virtual void | [**BeginPlay**](#function-beginplay) () override<br> |
| virtual void | [**EndPlay**](#function-endplay) (const EEndPlayReason::Type EndPlayReason) override<br> |
| virtual void | [**TickComponent**](#function-tickcomponent) (float DeltaTime, ELevelTick TickType, FActorComponentTickFunction \* ThisTickFunction) override<br> |




## Detailed Description


Moves a character's mouth with its voice. Plays the server's viseme track (the agent's voice profile needs Emit Visemes on) on a face with ARKit morph targets, in step with the audio the Tryll Speaker is playing.


Only the morph targets its pose table names are written; everything else on the mesh (blinks, brows, expressions) is left to the character's animation. Written morph targets override the AnimBP's value for that name while non-zero, and return to it at rest.


Requires a [**UTryllSpeakerComponent**](class_u_tryll_speaker_component.md): it is the audio, the playback clock and the source of the viseme track. 


    
## Public Attributes Documentation




### variable FaceMesh 

```C++
TObjectPtr<USkeletalMeshComponent> UTryllLipSyncComponent::FaceMesh;
```



Mesh carrying the ARKit morph targets. If unset, the owner's skeletal meshes are searched for one that has them. 


        

<hr>



### variable Poses 

```C++
TArray<FTryllVisemePose> UTryllLipSyncComponent::Poses;
```



Pose per viseme, as ARKit channel weights. Empty uses the built-in table. Read at BeginPlay. 


        

<hr>



### variable Settings 

```C++
FTryllVisemeSettings UTryllLipSyncComponent::Settings;
```



Look and sync. Live-editable in Play. 


        

<hr>



### variable Speaker 

```C++
TObjectPtr<UTryllSpeakerComponent> UTryllLipSyncComponent::Speaker;
```



Speaker whose TTS drives the face. Auto-discovered on the owner if unset. 


        

<hr>
## Public Functions Documentation




### function BindFace 

```C++
void UTryllLipSyncComponent::BindFace (
    USkeletalMeshComponent * InFaceMesh
) 
```



Rebinds to a face mesh (e.g. after spawning or swapping it at runtime). 


        

<hr>



### function GetActiveViseme 

```C++
inline ETryllViseme UTryllLipSyncComponent::GetActiveViseme () const
```




<hr>



### function GetAppliedJawOpen 

```C++
inline float UTryllLipSyncComponent::GetAppliedJawOpen () const
```



jawOpen as written this frame (0..1). 


        

<hr>



### function GetBoundChannelCount 

```C++
inline int32 UTryllLipSyncComponent::GetBoundChannelCount () const
```



Number of pose-table channels found as morph targets on the face. 


        

<hr>



### function GetPlayer 

```C++
inline const FTryllVisemePlayer & UTryllLipSyncComponent::GetPlayer () const
```



The player, for integrations that apply the curves themselves (e.g. a MetaHuman face). 


        

<hr>



### function IsFaceBound 

```C++
inline bool UTryllLipSyncComponent::IsFaceBound () const
```



True once at least one morph target in the pose table was found on the face. 


        

<hr>



### function SetIntensity 

```C++
inline void UTryllLipSyncComponent::SetIntensity (
    float InIntensity
) 
```



Convenience for the most-used knob. 


        

<hr>



### function UTryllLipSyncComponent 

```C++
UTryllLipSyncComponent::UTryllLipSyncComponent () 
```




<hr>
## Protected Functions Documentation




### function BeginPlay 

```C++
virtual void UTryllLipSyncComponent::BeginPlay () override
```




<hr>



### function EndPlay 

```C++
virtual void UTryllLipSyncComponent::EndPlay (
    const EEndPlayReason::Type EndPlayReason
) override
```




<hr>



### function TickComponent 

```C++
virtual void UTryllLipSyncComponent::TickComponent (
    float DeltaTime,
    ELevelTick TickType,
    FActorComponentTickFunction * ThisTickFunction
) override
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllLipSyncComponent.h`

