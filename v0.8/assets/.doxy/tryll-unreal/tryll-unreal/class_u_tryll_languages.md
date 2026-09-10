

# Class UTryllLanguages



[**ClassList**](annotated.md) **>** [**UTryllLanguages**](class_u_tryll_languages.md)



[More...](#detailed-description)

* `#include <TryllLanguages.h>`



Inherits the following classes: UBlueprintFunctionLibrary




































## Public Static Functions

| Type | Name |
| ---: | :--- |
|  bool | [**HasLanguageInfo**](#function-haslanguageinfo) (const TArray&lt; [**FTryllModelInfo**](struct_f_tryll_model_info.md) &gt; & Models) <br> |
|  FString | [**LanguageDisplayName**](#function-languagedisplayname) (const FString & Code) <br> |
|  FString | [**LanguageOf**](#function-languageof) (const FString & Locale) <br> |
|  bool | [**ModelSupportsLanguage**](#function-modelsupportslanguage) (const [**FTryllModelInfo**](struct_f_tryll_model_info.md) & Model, const FString & Locale) <br> |
|  TArray&lt; FString &gt; | [**OfferableLanguages**](#function-offerablelanguages) (const TArray&lt; [**FTryllModelInfo**](struct_f_tryll_model_info.md) &gt; & Models) <br> |
|  TArray&lt; FString &gt; | [**OfferableLanguagesForModels**](#function-offerablelanguagesformodels) (const TArray&lt; [**FTryllModelInfo**](struct_f_tryll_model_info.md) &gt; & AllModels, const TArray&lt; FString &gt; & ModelNames) <br> |


























## Detailed Description


Works out which languages your game can actually offer, from the models it actually uses.


The rule is an intersection: a language is offerable only if _every_ model in the pipeline covers it. A game with a 31-language voice but an English-only recognizer can offer exactly one language for hands-free play - and the point of this library is that you find that out from the catalog instead of from a bug report.


Blueprint-callable, so a settings menu can populate itself without C++. 


    
## Public Static Functions Documentation




### function HasLanguageInfo 

```C++
static bool UTryllLanguages::HasLanguageInfo (
    const TArray< FTryllModelInfo > & Models
) 
```



True when at least one model declared its languages. Distinguishes "these
models share no language" from "we have no idea", which are very different things to show a player. 


        

<hr>



### function LanguageDisplayName 

```C++
static FString UTryllLanguages::LanguageDisplayName (
    const FString & Code
) 
```



English name for a language code - "de" -&gt; "German" - for labelling a menu. Unknown codes come back unchanged, so an entry is never blank.


Deliberately a fixed table rather than FInternationalization: it must match what the server renders into the {{language}} prompt key exactly (see SessionLocale.cpp). Keep the two lists in sync when adding a language. 


        

<hr>



### function LanguageOf 

```C++
static FString UTryllLanguages::LanguageOf (
    const FString & Locale
) 
```



Primary language subtag of a locale, lowercase: "pt-BR" -&gt; "pt". This is the granularity models work at - none of the shipped voice or speech models distinguish pt-BR from pt-PT. 


        

<hr>



### function ModelSupportsLanguage 

```C++
static bool UTryllLanguages::ModelSupportsLanguage (
    const FTryllModelInfo & Model,
    const FString & Locale
) 
```



Whether Model covers Locale (a tag like "de-DE" or a bare code like "de").


Returns true for a model that declares nothing: without a catalog claim we have no evidence either way, and the server applies the same rule - it withholds the language hint rather than refusing to load the model. 


        

<hr>



### function OfferableLanguages 

```C++
static TArray< FString > UTryllLanguages::OfferableLanguages (
    const TArray< FTryllModelInfo > & Models
) 
```



Languages every model in Models declares, sorted.


A model that declares nothing is **skipped, not treated as restrictive** - an empty list means "the catalog didn't say", and letting unknown shrink the result would hide languages that work.


An empty return therefore means one of two things: the declared sets genuinely do not overlap, or nothing declared anything. Call HasLanguageInfo to tell those apart before telling a player there are no languages available. 


        

<hr>



### function OfferableLanguagesForModels 

```C++
static TArray< FString > UTryllLanguages::OfferableLanguagesForModels (
    const TArray< FTryllModelInfo > & AllModels,
    const TArray< FString > & ModelNames
) 
```



Same, but picks the models out of a ListModels result by name - the usual shape, since you know your TTS/STT/language model names. Names that are not present are ignored, so a model the player has not downloaded yet does not silently empty the menu. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllLanguages.h`

