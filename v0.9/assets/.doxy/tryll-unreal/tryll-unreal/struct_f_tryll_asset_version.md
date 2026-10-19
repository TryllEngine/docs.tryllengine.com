

# Struct FTryllAssetVersion



[**ClassList**](annotated.md) **>** [**FTryllAssetVersion**](struct_f_tryll_asset_version.md)



[More...](#detailed-description)

* `#include <TryllAssetVersion.h>`

















## Public Types

| Type | Name |
| ---: | :--- |
| enum  | [**Type**](#enum-type)  <br> |






## Public Static Attributes

| Type | Name |
| ---: | :--- |
|  const FGuid | [**Guid**](#variable-guid)  <br> |










































## Detailed Description


Saved into every package that holds Tryll node params. Packages saved before it existed read as -1. One entry per change that needs a conversion in TryllMigrations.cpp. Append only: never remove or reorder entries, or saved assets will skip conversions or fail to load. 


    
## Public Types Documentation




### enum Type 

```C++
enum FTryllAssetVersion::Type {
    BeforeCustomVersionWasAdded = 0,
    ClassifyIntentLLMIntentsList,
    VersionPlusOne,
    LatestVersion = VersionPlusOne - 1
};
```




<hr>
## Public Static Attributes Documentation




### variable Guid 

```C++
const FGuid FTryllAssetVersion::Guid;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllAssetVersion.h`

