

# File TryllVoiceInputComponent.h



[**FileList**](files.md) **>** [**client-unreal**](dir_95d666eee9112f2bfa7f2b736b6243b9.md) **>** [**Source**](dir_e37f8a870dc803113e92bf135247a735.md) **>** [**TryllClient**](dir_0869abba98a308e3c3eadd7e169e0f62.md) **>** [**Public**](dir_338741d27b4bda5805009de80ddaf6fc.md) **>** [**TryllVoiceInputComponent.h**](_tryll_voice_input_component_8h.md)





* `#include "CoreMinimal.h"`
* `#include "Components/ActorComponent.h"`
* `#include "TryllError.h"`
* `#include "TryllVoiceInput.h"`
* `#include "TryllVoiceInputComponent.generated.h"`





































## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE**](#function-declare_dynamic_multicast_delegate) (FOnTryllVoiceInputCreated) <br> |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_OneParam**](#function-declare_dynamic_multicast_delegate_oneparam) (FOnTryllTranscriptUpdate, const [**FTryllTranscriptUpdate**](struct_f_tryll_transcript_update.md) &, Update) <br> |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_OneParam**](#function-declare_dynamic_multicast_delegate_oneparam) (FOnTryllVoiceError, const FString &, ErrorMessage) <br> |




























## Public Functions Documentation




### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE (
    FOnTryllVoiceInputCreated
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_OneParam 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam (
    FOnTryllTranscriptUpdate,
    const FTryllTranscriptUpdate &,
    Update
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_OneParam 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam (
    FOnTryllVoiceError,
    const FString &,
    ErrorMessage
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/TryllVoiceInputComponent.h`

