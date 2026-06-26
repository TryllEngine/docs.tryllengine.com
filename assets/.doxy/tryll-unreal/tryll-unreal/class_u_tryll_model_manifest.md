

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
|  ETryllModelRegistration | [**Get**](#function-get) (const FString & ModelName) const<br> |
| virtual FName | [**GetCategoryName**](#function-getcategoryname) () override const<br> |
|  TArray&lt; FString &gt; | [**ModelsOfType**](#function-modelsoftype) (const FString & ModelType) const<br> |
|  void | [**Set**](#function-set) (const FString & ModelName, const FString & ModelType, ETryllModelRegistration Registration) <br> |




























## Detailed Description


Project-wide manifest of which models ship in the build. Stores a thin name -&gt; { type, registration } list; catalog metadata stays server-owned and is read via ListModels. The Model Manager window writes it; component pickers and a future packaging step read it.


Persisted to &lt;YourProject&gt;/Config/DefaultGame.ini under [/Script/TryllClient.TryllModelManifest]. 


    
## Public Attributes Documentation




### variable Entries 

```C++
TArray<FTryllModelManifestEntry> UTryllModelManifest::Entries;
```



Models the project has registered. Unlisted models are None. 


        

<hr>
## Public Functions Documentation




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
inline void UTryllModelManifest::Set (
    const FString & ModelName,
    const FString & ModelType,
    ETryllModelRegistration Registration
) 
```



Set (or clear) a model's registration. Setting None removes the entry so the manifest only holds models the project actually selects. Does not persist; callers invoke TryUpdateDefaultConfigFile() to write DefaultGame.ini. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllModelManifest.h`

