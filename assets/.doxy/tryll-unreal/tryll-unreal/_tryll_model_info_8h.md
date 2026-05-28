

# File TryllModelInfo.h



[**FileList**](files.md) **>** [**client-unreal**](dir_95d666eee9112f2bfa7f2b736b6243b9.md) **>** [**Source**](dir_e37f8a870dc803113e92bf135247a735.md) **>** [**TryllClient**](dir_0869abba98a308e3c3eadd7e169e0f62.md) **>** [**Public**](dir_338741d27b4bda5805009de80ddaf6fc.md) **>** [**TryllModelInfo.h**](_tryll_model_info_8h.md)





* `#include "CoreMinimal.h"`
* `#include "TryllModelInfo.generated.h"`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTryllModelInfo**](struct_f_tryll_model_info.md) <br> |


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
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/TryllModelInfo.h`

