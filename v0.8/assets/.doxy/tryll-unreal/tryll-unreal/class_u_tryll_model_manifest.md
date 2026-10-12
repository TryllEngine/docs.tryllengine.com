

# Class UTryllModelManifest



[**ClassList**](annotated.md) **>** [**UTryllModelManifest**](class_u_tryll_model_manifest.md)



[More...](#detailed-description)

* `#include <TryllModelManifest.h>`



Inherits the following classes: UDeveloperSettings


















## Public Attributes

| Type | Name |
| ---: | :--- |
|  TArray&lt; [**FTryllModelManifestEntry**](struct_f_tryll_model_manifest_entry.md) &gt; | [**Entries**](#variable-entries)  <br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  TArray&lt; FString &gt; | [**DeclaredLanguages**](#function-declaredlanguages) (const TArray&lt; FString &gt; & ModelTypes={}) const<br> |
|  ETryllModelRegistration | [**Get**](#function-get) (const FString & ModelName) const<br> |
| virtual FName | [**GetCategoryName**](#function-getcategoryname) () override const<br> |
|  TArray&lt; FString &gt; | [**LanguagesOf**](#function-languagesof) (const FString & ModelName) const<br> |
|  TArray&lt; FString &gt; | [**ModelsMissingLanguage**](#function-modelsmissinglanguage) (const FString & Language, const TArray&lt; FString &gt; & ModelTypes={}) const<br> |
|  TArray&lt; FString &gt; | [**ModelsOfType**](#function-modelsoftype) (const FString & ModelType) const<br> |
|  bool | [**Set**](#function-set) (const FString & ModelName, const FString & ModelType, ETryllModelRegistration Registration, const TArray&lt; FString &gt; \* Languages=nullptr, const ETryllInferenceEngine \* Engine=nullptr, const ETryllMemoryPlacement \* Placement=nullptr, const ETryllHotwordBias \* Hotwords=nullptr) <br> |




























## Detailed Description


Project-wide manifest of which models ship in the build. Stores a thin name -&gt; { type, registration, languages } list; the rest of the catalog stays server-owned and is read via ListModels. The Model Manager window writes it; component pickers and a future packaging step read it.


Persisted to &lt;YourProject&gt;/Config/DefaultGame.ini under [/Script/TryllClient.TryllModelManifest]. 


    
## Public Attributes Documentation




### variable Entries 

```C++
TArray<FTryllModelManifestEntry> UTryllModelManifest::Entries;
```



Models the project has registered. Unlisted models are None. 


        

<hr>
## Public Functions Documentation




### function DeclaredLanguages 

```C++
inline TArray< FString > UTryllModelManifest::DeclaredLanguages (
    const TArray< FString > & ModelTypes={}
) const
```



Every language any registered model declares, sorted - the source for the settings language dropdown.


A **union**, deliberately, not an intersection: a project may register an English-only model for one character and a 31-language voice for everyone else, and intersecting would collapse the menu to English. Per-model mismatches are surfaced where they are actionable instead - on the model picker (inspector warning) and at agent creation (server error).


Empty ModelTypes means "all types". 


        

<hr>



### function Get 

```C++
inline ETryllModelRegistration UTryllModelManifest::Get (
    const FString & ModelName
) const
```



Registration for a model, or None if unlisted. 


        

<hr>



### function GetCategoryName 

```C++
inline virtual FName UTryllModelManifest::GetCategoryName () override const
```




<hr>



### function LanguagesOf 

```C++
inline TArray< FString > UTryllModelManifest::LanguagesOf (
    const FString & ModelName
) const
```



Cached languages for a registered model; empty when unlisted or undeclared. 


        

<hr>



### function ModelsMissingLanguage 

```C++
inline TArray< FString > UTryllModelManifest::ModelsMissingLanguage (
    const FString & Language,
    const TArray< FString > & ModelTypes={}
) const
```



Registered models of the given types that do NOT declare Language, ignoring those that declared nothing. Drives the "N voice models cannot do German" summary in project settings. 


        

<hr>



### function ModelsOfType 

```C++
inline TArray< FString > UTryllModelManifest::ModelsOfType (
    const FString & ModelType
) const
```



Names of registered models of a given type, for type-filtered pickers. 


        

<hr>



### function Set 

```C++
inline bool UTryllModelManifest::Set (
    const FString & ModelName,
    const FString & ModelType,
    ETryllModelRegistration Registration,
    const TArray< FString > * Languages=nullptr,
    const ETryllInferenceEngine * Engine=nullptr,
    const ETryllMemoryPlacement * Placement=nullptr,
    const ETryllHotwordBias * Hotwords=nullptr
) 
```



Set (or clear) a model's registration. Setting None removes the entry so the manifest only holds models the project actually selects. Does not persist; callers invoke TryUpdateDefaultConfigFile() to write DefaultGame.ini. Returns true when this call actually changed the manifest, so a caller knows whether to write DefaultGame.ini.


Reported from here rather than compared by the caller, and that is the point. Callers used to decide for themselves — the Model Manager's catalog refresh skipped the whole call whenever cached _languages_ matched the server, which meant a stale cached _engine_ was never corrected. Every manifest written before the engine field existed therefore kept the enum default and silently pre-warmed nothing. Only this method knows what it caches, so only this method can say whether a write mattered; adding a field later cannot reintroduce the same bug. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllModelManifest.h`

