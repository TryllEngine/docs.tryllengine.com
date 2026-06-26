

# File TryllModelInfo.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllModelInfo.h**](_tryll_model_info_8h.md)





* `#include "CoreMinimal.h"`
* `#include "TryllModelInfo.generated.h"`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTryllModelInfo**](struct_f_tryll_model_info.md) <br> |
| struct | [**FTryllSamplingParams**](struct_f_tryll_sampling_params.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| enum uint8 | [**ETryllModelStatus**](#enum-etryllmodelstatus)  <br> |
















































## Public Types Documentation




### enum ETryllModelStatus 

```C++
enum ETryllModelStatus {
    Absent = 0,
    Local = 1,
    Downloading = 2,
    Loaded = 3,
    Downloaded = 4
};
```



Status of a model as reported by the server. Mirrors the FlatBuffers ModelStatus enum in messages.fbs — ordinal values must stay in sync. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllModelInfo.h`

