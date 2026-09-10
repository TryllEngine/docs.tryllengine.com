

# Struct FTryllModelManifestEntry



[**ClassList**](annotated.md) **>** [**FTryllModelManifestEntry**](struct_f_tryll_model_manifest_entry.md)



[More...](#detailed-description)

* `#include <TryllModelManifest.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  ETryllInferenceEngine | [**Engine**](#variable-engine)   = `ETryllInferenceEngine::LlamaCpp`<br> |
|  TArray&lt; FString &gt; | [**Languages**](#variable-languages)  <br> |
|  FString | [**ModelType**](#variable-modeltype)  <br> |
|  FString | [**Name**](#variable-name)  <br> |
|  ETryllMemoryPlacement | [**Placement**](#variable-placement)   = `ETryllMemoryPlacement::Auto`<br> |
|  ETryllModelRegistration | [**Registration**](#variable-registration)   = `ETryllModelRegistration::None`<br> |












































## Detailed Description


One registered model: its name, type, and build tier. 


    
## Public Attributes Documentation




### variable Engine 

```C++
ETryllInferenceEngine FTryllModelManifestEntry::Engine;
```



Engine that runs this model, cached from the catalog when it was registered.


Cached rather than read live because the session's engine set is built from this manifest at CreateSession, before any ListModels round trip could answer — an editor that could only build a session while connected would be worse than none. Defaults to LlamaCpp, not Mock: Mock is the enum's zero value and a production server refuses it. 


        

<hr>



### variable Languages 

```C++
TArray<FString> FTryllModelManifestEntry::Languages;
```



BCP-47 primary language subtags this model declares, cached from the catalog.


The one piece of catalog metadata cached here rather than read live. Everything else can wait for a ListModels round trip, but the settings language dropdown and the model-picker warnings have to work with no server running - an editor that can only validate languages while connected would be worse than no validation at all. Written by the Model Manager from the live [**FTryllModelInfo**](struct_f_tryll_model_info.md) at the moment the model is registered.


Empty means undeclared, not "supports nothing". 


        

<hr>



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



### variable Placement 

```C++
ETryllMemoryPlacement FTryllModelManifestEntry::Placement;
```



Where this model should live. See ETryllMemoryPlacement. 


        

<hr>



### variable Registration 

```C++
ETryllModelRegistration FTryllModelManifestEntry::Registration;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllModelManifest.h`

