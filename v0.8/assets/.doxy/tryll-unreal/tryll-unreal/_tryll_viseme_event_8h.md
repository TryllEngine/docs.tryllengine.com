

# File TryllVisemeEvent.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllVisemeEvent.h**](_tryll_viseme_event_8h.md)





* `#include "CoreMinimal.h"`
* `#include "TryllVisemeEvent.generated.h"`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTryllVisemeChannelWeight**](struct_f_tryll_viseme_channel_weight.md) <br> |
| struct | [**FTryllVisemeEvent**](struct_f_tryll_viseme_event.md) <br> |
| struct | [**FTryllVisemePose**](struct_f_tryll_viseme_pose.md) <br> |
| struct | [**FTryllVisemeSettings**](struct_f_tryll_viseme_settings.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| enum uint8 | [**ETryllViseme**](#enum-etryllviseme)  <br> |
















































## Public Types Documentation




### enum ETryllViseme 

```C++
enum ETryllViseme {
    sil = 0,
    PP = 1,
    FF = 2,
    TH = 3,
    DD = 4,
    kk = 5,
    CH = 6,
    SS = 7,
    nn = 8,
    RR = 9,
    aa = 10,
    E = 11,
    ih = 12,
    oh = 13,
    ou = 14
};
```



Oculus 15-viseme set, as sent in TtsVisemeFrame. Mirrors the FlatBuffers WireViseme enum in messages.fbs — ordinal values must stay in sync. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllVisemeEvent.h`

