

# File TryllModelManifest.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllModelManifest.h**](_tryll_model_manifest_8h.md)





* `#include "CoreMinimal.h"`
* `#include "Engine/DeveloperSettings.h"`
* `#include "TryllGraphDescription.h"`
* `#include "TryllModelManifest.generated.h"`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTryllModelManifestEntry**](struct_f_tryll_model_manifest_entry.md) <br> |
| class | [**UTryllModelManifest**](class_u_tryll_model_manifest.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| enum uint8 | [**ETryllMemoryPlacement**](#enum-etryllmemoryplacement)  <br> |
| enum uint8 | [**ETryllModelRegistration**](#enum-etryllmodelregistration)  <br> |
















































## Public Types Documentation




### enum ETryllMemoryPlacement 

```C++
enum ETryllMemoryPlacement {
    Auto = 0,
    Ram = 1,
    Vram = 2
};
```



Per-model build selection for this project. Ordinals match the Unity client's TryllModelRegistration so a shared manifest stays interchangeable. Where a model's weights should live. Mirrors the wire MemoryPlacement enum.


Always a hint: a machine that cannot honour it loads the model the other way and reports what it did. That matters here because this value is authored on a developer's machine and shipped to players' machines — a VRAM choice that suits a 24 GB card is not a promise an 8 GB card can keep.


Which GPU API provides VRAM (CUDA / Vulkan) is deliberately not expressible: that is the host's decision, not the project's. 


        

<hr>



### enum ETryllModelRegistration 

```C++
enum ETryllModelRegistration {
    None = 0,
    Experimental = 1,
    Production = 2
};
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllModelManifest.h`

