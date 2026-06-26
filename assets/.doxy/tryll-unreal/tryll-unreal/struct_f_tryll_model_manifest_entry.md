

# Struct FTryllModelManifestEntry



[**ClassList**](annotated.md) **>** [**FTryllModelManifestEntry**](struct_f_tryll_model_manifest_entry.md)



[More...](#detailed-description)

* `#include <TryllModelManifest.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**ModelType**](#variable-modeltype)  <br> |
|  FString | [**Name**](#variable-name)  <br> |
|  ETryllModelRegistration | [**Registration**](#variable-registration)   = `ETryllModelRegistration::None`<br> |












































## Detailed Description


One registered model: its name, type, and build tier. 


    
## Public Attributes Documentation




### variable ModelType 

```C++
FString FTryllModelManifestEntry::ModelType;
```



"language" \| "embedding" \| "stt" \| "tts" \| "vad" — for type-filtered pickers. 


        

<hr>



### variable Name 

```C++
FString FTryllModelManifestEntry::Name;
```




<hr>



### variable Registration 

```C++
ETryllModelRegistration FTryllModelManifestEntry::Registration;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllModelManifest.h`

