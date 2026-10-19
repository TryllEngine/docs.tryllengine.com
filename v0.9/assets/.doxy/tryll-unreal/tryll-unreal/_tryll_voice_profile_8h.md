

# File TryllVoiceProfile.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllVoiceProfile.h**](_tryll_voice_profile_8h.md)





* `#include "CoreMinimal.h"`
* `#include "Engine/DataAsset.h"`
* `#include "TryllModelName.h"`
* `#include "TryllStoragePath.h"`
* `#include "TryllVoiceProfile.generated.h"`













## Namespaces

| Type | Name |
| ---: | :--- |
| namespace | [**Tryll**](namespace_tryll.md) <br> |


## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTryllDeliveryMode**](struct_f_tryll_delivery_mode.md) <br> |
| struct | [**FTryllVoiceProfileData**](struct_f_tryll_voice_profile_data.md) <br> |
| class | [**UTryllVoiceProfile**](class_u_tryll_voice_profile.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| enum uint8 | [**ETryllVoicePreparation**](#enum-etryllvoicepreparation)  <br> |
















































## Public Types Documentation




### enum ETryllVoicePreparation 

```C++
enum ETryllVoicePreparation {
    Eager = 0,
    Lazy = 1
};
```



When the server prepares a voice's reference clips (speaker embeddings, decoded audio). Mirrors Tryll::VoicePreparation in voice.fbs. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllVoiceProfile.h`

