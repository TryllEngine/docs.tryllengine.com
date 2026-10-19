

# Class UTryllStringTableTextProvider



[**ClassList**](annotated.md) **>** [**UTryllStringTableTextProvider**](class_u_tryll_string_table_text_provider.md)



[More...](#detailed-description)

* `#include <TryllStringTableTextProvider.h>`



Inherits the following classes: [UTryllTextProvider](class_u_tryll_text_provider.md)






















































## Public Functions

| Type | Name |
| ---: | :--- |
| virtual void | [**BeginDestroy**](#function-begindestroy) () override<br> |
| virtual bool | [**GetTextForPreview**](#function-gettextforpreview) (FName Table, const FString & Id, const FString & Locale, FString & OutText) override const<br> |
| virtual void | [**InvalidatePreviewCache**](#function-invalidatepreviewcache) () override<br> |
| virtual TArray&lt; FString &gt; | [**ListIds**](#function-listids) (FName Table) override const<br> |
| virtual TArray&lt; FString &gt; | [**ListLocales**](#function-listlocales) () override const<br> |
| virtual bool | [**OpenInEditor**](#function-openineditor) (FName Table, const FString & Id, FString & OutMessage) override const<br> |
| virtual void | [**PostInitProperties**](#function-postinitproperties) () override<br> |
| virtual bool | [**SupportsEditorPreview**](#function-supportseditorpreview) () override const<br> |


## Public Functions inherited from UTryllTextProvider

See [UTryllTextProvider](class_u_tryll_text_provider.md)

| Type | Name |
| ---: | :--- |
|  FString | [**GetCurrentLocale**](class_u_tryll_text_provider.md#function-getcurrentlocale) () const<br> |
|  bool | [**GetText**](class_u_tryll_text_provider.md#function-gettext) (FName Table, const FString & Id, FString & OutText) const<br> |
| virtual bool | [**GetTextForPreview**](class_u_tryll_text_provider.md#function-gettextforpreview) (FName Table, const FString & Id, const FString & Locale, FString & OutText) const<br> |
| virtual void | [**InvalidatePreviewCache**](class_u_tryll_text_provider.md#function-invalidatepreviewcache) () <br> |
| virtual TArray&lt; FString &gt; | [**ListIds**](class_u_tryll_text_provider.md#function-listids) (FName Table) const<br> |
| virtual TArray&lt; FString &gt; | [**ListLocales**](class_u_tryll_text_provider.md#function-listlocales) () const<br> |
| virtual bool | [**OpenInEditor**](class_u_tryll_text_provider.md#function-openineditor) (FName Table, const FString & Id, FString & OutMessage) const<br> |
| virtual bool | [**SupportsEditorPreview**](class_u_tryll_text_provider.md#function-supportseditorpreview) () const<br> |


## Public Static Functions

| Type | Name |
| ---: | :--- |
|  bool | [**FindOrLoadTable**](#function-findorloadtable) (FName TableId, FName & OutResolvedId) <br> |


## Public Static Functions inherited from UTryllTextProvider

See [UTryllTextProvider](class_u_tryll_text_provider.md)

| Type | Name |
| ---: | :--- |
|  void | [**BroadcastLocaleChanged**](class_u_tryll_text_provider.md#function-broadcastlocalechanged) () <br> |
|  FString | [**CurrentProviderLocale**](class_u_tryll_text_provider.md#function-currentproviderlocale) () <br> |
|  [**UTryllTextProvider**](class_u_tryll_text_provider.md) \* | [**Get**](class_u_tryll_text_provider.md#function-get) () <br> |
|  bool | [**LocaleForLanguageChange**](class_u_tryll_text_provider.md#function-localeforlanguagechange) (bool bSessionLocaleExplicit, FString & OutLocale) <br> |
|  FSimpleMulticastDelegate & | [**OnLocaleChanged**](class_u_tryll_text_provider.md#function-onlocalechanged) () <br> |
|  FString | [**ResolveSessionLocale**](class_u_tryll_text_provider.md#function-resolvesessionlocale) () <br> |










































## Protected Functions

| Type | Name |
| ---: | :--- |
| virtual FString | [**GetCurrentLocale\_Implementation**](#function-getcurrentlocale_implementation) () override const<br> |
| virtual bool | [**GetText\_Implementation**](#function-gettext_implementation) (FName Table, const FString & Id, FString & OutText) override const<br> |








## Detailed Description


Reads Unreal string tables (FText::FromStringTable) and follows the game culture. Opt-in: pick it in Project Settings → Tryll Client → Text Provider.


Smart-string formatting is not applied: the table text is sent as authored, so `{}` is not nested inside Mustache `{{ }}`.


A table that is not loaded yet is loaded on first use (blocking, game thread). The language-change signal fires after Unreal has loaded the new language's text (FTextLocalizationManager::OnTextRevisionChangedEvent), never before. 


    
## Public Functions Documentation




### function BeginDestroy 

```C++
virtual void UTryllStringTableTextProvider::BeginDestroy () override
```




<hr>



### function GetTextForPreview 

```C++
virtual bool UTryllStringTableTextProvider::GetTextForPreview (
    FName Table,
    const FString & Id,
    const FString & Locale,
    FString & OutText
) override const
```



Exact text for Locale, read from that culture's compiled .locres files without changing the editor or game culture. The native culture reads the table source. False when Locale has no translation (or only a stale one) for the entry. 


        
Implements [*UTryllTextProvider::GetTextForPreview*](class_u_tryll_text_provider.md#function-gettextforpreview)


<hr>



### function InvalidatePreviewCache 

```C++
virtual void UTryllStringTableTextProvider::InvalidatePreviewCache () override
```



Drop cached preview data (table edits, recompiled translations). 


        
Implements [*UTryllTextProvider::InvalidatePreviewCache*](class_u_tryll_text_provider.md#function-invalidatepreviewcache)


<hr>



### function ListIds 

```C++
virtual TArray< FString > UTryllStringTableTextProvider::ListIds (
    FName Table
) override const
```



Implements [*UTryllTextProvider::ListIds*](class_u_tryll_text_provider.md#function-listids)


<hr>



### function ListLocales 

```C++
virtual TArray< FString > UTryllStringTableTextProvider::ListLocales () override const
```



Implements [*UTryllTextProvider::ListLocales*](class_u_tryll_text_provider.md#function-listlocales)


<hr>



### function OpenInEditor 

```C++
virtual bool UTryllStringTableTextProvider::OpenInEditor (
    FName Table,
    const FString & Id,
    FString & OutMessage
) override const
```



Jump to the table entry in the editor. False with OutMessage when the table has no editor. 


        
Implements [*UTryllTextProvider::OpenInEditor*](class_u_tryll_text_provider.md#function-openineditor)


<hr>



### function PostInitProperties 

```C++
virtual void UTryllStringTableTextProvider::PostInitProperties () override
```




<hr>



### function SupportsEditorPreview 

```C++
inline virtual bool UTryllStringTableTextProvider::SupportsEditorPreview () override const
```



Editor preview. The string-table provider supports it. A Blueprint-only provider returns false and the details panel shows no preview. 


        
Implements [*UTryllTextProvider::SupportsEditorPreview*](class_u_tryll_text_provider.md#function-supportseditorpreview)


<hr>
## Public Static Functions Documentation




### function FindOrLoadTable 

```C++
static bool UTryllStringTableTextProvider::FindOrLoadTable (
    FName TableId,
    FName & OutResolvedId
) 
```



Find a registered string table, loading its asset when needed. Game thread only. 


        

<hr>
## Protected Functions Documentation




### function GetCurrentLocale\_Implementation 

```C++
virtual FString UTryllStringTableTextProvider::GetCurrentLocale_Implementation () override const
```




<hr>



### function GetText\_Implementation 

```C++
virtual bool UTryllStringTableTextProvider::GetText_Implementation (
    FName Table,
    const FString & Id,
    FString & OutText
) override const
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllStringTableTextProvider.h`

