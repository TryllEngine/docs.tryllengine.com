

# Struct FTryllStoragePath



[**ClassList**](annotated.md) **>** [**FTryllStoragePath**](struct_f_tryll_storage_path.md)



[More...](#detailed-description)

* `#include <TryllStoragePath.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Path**](#variable-path)  <br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  bool | [**operator==**](#function-operator) (const [**FTryllStoragePath**](struct_f_tryll_storage_path.md) & Other) const<br> |




























## Detailed Description


Path relative to TryllRuntimeSettings::StorageDataFolder (the session storage\_root), using forward slashes — e.g. "RAG/lore.json", not "StringStorages/RAG/lore.json". Editor UI: browse + validation via FTryllStoragePathCustomization (TryllClientEditor). 


    
## Public Attributes Documentation




### variable Path 

```C++
FString FTryllStoragePath::Path;
```




<hr>
## Public Functions Documentation




### function operator== 

```C++
inline bool FTryllStoragePath::operator== (
    const FTryllStoragePath & Other
) const
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllStoragePath.h`

