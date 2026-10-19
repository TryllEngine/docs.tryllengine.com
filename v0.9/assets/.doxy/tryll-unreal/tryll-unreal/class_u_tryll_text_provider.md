

# Class UTryllTextProvider



[**ClassList**](annotated.md) **>** [**UTryllTextProvider**](class_u_tryll_text_provider.md)



[More...](#detailed-description)

* `#include <TryllTextProvider.h>`



Inherits the following classes: UObject


Inherited by the following classes: [UTryllStringTableTextProvider](class_u_tryll_string_table_text_provider.md)
































## Public Functions

| Type | Name |
| ---: | :--- |
|  FString | [**GetCurrentLocale**](#function-getcurrentlocale) () const<br> |
|  bool | [**GetText**](#function-gettext) (FName Table, const FString & Id, FString & OutText) const<br> |
| virtual bool | [**GetTextForPreview**](#function-gettextforpreview) (FName Table, const FString & Id, const FString & Locale, FString & OutText) const<br> |
| virtual void | [**InvalidatePreviewCache**](#function-invalidatepreviewcache) () <br> |
| virtual TArray&lt; FString &gt; | [**ListIds**](#function-listids) (FName Table) const<br> |
| virtual TArray&lt; FString &gt; | [**ListLocales**](#function-listlocales) () const<br> |
| virtual bool | [**OpenInEditor**](#function-openineditor) (FName Table, const FString & Id, FString & OutMessage) const<br> |
| virtual bool | [**SupportsEditorPreview**](#function-supportseditorpreview) () const<br> |


## Public Static Functions

| Type | Name |
| ---: | :--- |
|  void | [**BroadcastLocaleChanged**](#function-broadcastlocalechanged) () <br> |
|  FString | [**CurrentProviderLocale**](#function-currentproviderlocale) () <br> |
|  [**UTryllTextProvider**](class_u_tryll_text_provider.md) \* | [**Get**](#function-get) () <br> |
|  bool | [**LocaleForLanguageChange**](#function-localeforlanguagechange) (bool bSessionLocaleExplicit, FString & OutLocale) <br> |
|  FSimpleMulticastDelegate & | [**OnLocaleChanged**](#function-onlocalechanged) () <br> |
|  FString | [**ResolveSessionLocale**](#function-resolvesessionlocale) () <br> |


























## Detailed Description


Game-owned lookup for Text IDs on a workflow. Tryll does not own a string table. The class is chosen in Project Settings → Tryll Client → Text Provider. None means prompts stay inline and the Tryll Locale field is the session language. 


    
## Public Functions Documentation




### function GetCurrentLocale 

```C++
FString UTryllTextProvider::GetCurrentLocale () const
```



BCP-47 tag of the game's current language. Empty = unset. 


        

<hr>



### function GetText 

```C++
bool UTryllTextProvider::GetText (
    FName Table,
    const FString & Id,
    FString & OutText
) const
```



Text for Id in Table, in the provider's current language, after the provider's own fallback. False when the ID is unknown. 


        

<hr>



### function GetTextForPreview 

```C++
inline virtual bool UTryllTextProvider::GetTextForPreview (
    FName Table,
    const FString & Id,
    const FString & Locale,
    FString & OutText
) const
```



Exact locale, no fallback. False when that locale has no entry. 


        

<hr>



### function InvalidatePreviewCache 

```C++
inline virtual void UTryllTextProvider::InvalidatePreviewCache () 
```



Drop cached preview data (table edits, recompiled translations). 


        

<hr>



### function ListIds 

```C++
inline virtual TArray< FString > UTryllTextProvider::ListIds (
    FName Table
) const
```




<hr>



### function ListLocales 

```C++
inline virtual TArray< FString > UTryllTextProvider::ListLocales () const
```




<hr>



### function OpenInEditor 

```C++
inline virtual bool UTryllTextProvider::OpenInEditor (
    FName Table,
    const FString & Id,
    FString & OutMessage
) const
```



Jump to the table entry in the editor. False with OutMessage when the table has no editor. 


        

<hr>



### function SupportsEditorPreview 

```C++
inline virtual bool UTryllTextProvider::SupportsEditorPreview () const
```



Editor preview. The string-table provider supports it. A Blueprint-only provider returns false and the details panel shows no preview. 


        

<hr>
## Public Static Functions Documentation




### function BroadcastLocaleChanged 

```C++
static void UTryllTextProvider::BroadcastLocaleChanged () 
```




<hr>



### function CurrentProviderLocale 

```C++
static FString UTryllTextProvider::CurrentProviderLocale () 
```



The provider's current language, or empty when no provider is set. 


        

<hr>



### function Get 

```C++
static UTryllTextProvider * UTryllTextProvider::Get () 
```



The instance from the project settings class, or null when the class is None. 


        

<hr>



### function LocaleForLanguageChange 

```C++
static bool UTryllTextProvider::LocaleForLanguageChange (
    bool bSessionLocaleExplicit,
    FString & OutLocale
) 
```



Session language to send after a language change. False (send nothing) when there is no provider or CreateSession was given an explicit locale. An empty provider language falls back to [**UTryllRuntimeSettings::Locale**](class_u_tryll_runtime_settings.md#variable-locale), never "". 


        

<hr>



### function OnLocaleChanged 

```C++
static FSimpleMulticastDelegate & UTryllTextProvider::OnLocaleChanged () 
```



Raised after the game's language changed. Game and editor subsystems both listen. 


        

<hr>



### function ResolveSessionLocale 

```C++
static FString UTryllTextProvider::ResolveSessionLocale () 
```



Session language for an empty CreateSession locale: the provider's current language when one is set, otherwise [**UTryllRuntimeSettings::Locale**](class_u_tryll_runtime_settings.md#variable-locale). 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllTextProvider.h`

