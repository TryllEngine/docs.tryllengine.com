

# Struct FTryllTextPreview



[**ClassList**](annotated.md) **>** [**FTryllTextPreview**](struct_f_tryll_text_preview.md)



[More...](#detailed-description)

* `#include <TryllTextProvider.h>`

















## Public Types

| Type | Name |
| ---: | :--- |
| enum uint8 | [**EKind**](#enum-ekind)  <br> |




## Public Attributes

| Type | Name |
| ---: | :--- |
|  EKind | [**Kind**](#variable-kind)   = `EKind::NoEditorSupport`<br> |
|  FString | [**Message**](#variable-message)  <br> |
|  TArray&lt; FString &gt; | [**MissingLocales**](#variable-missinglocales)  <br> |
|  FString | [**Text**](#variable-text)  <br> |
|  bool | [**bIsFallback**](#variable-bisfallback)   = `false`<br> |


















## Public Static Functions

| Type | Name |
| ---: | :--- |
|  [**FTryllTextPreview**](struct_f_tryll_text_preview.md) | [**Evaluate**](#function-evaluate) (const [**UTryllTextProvider**](class_u_tryll_text_provider.md) \* Provider, FName Table, const FString & Id, const FString & PreviewLocale) <br> |


























## Detailed Description


Inspector preview for one Text ID. Shared by the editor and automation tests. 


    
## Public Types Documentation




### enum EKind 

```C++
enum FTryllTextPreview::EKind {
    Found,
    MissingHere,
    EmptyId,
    NoEditorSupport
};
```




<hr>
## Public Attributes Documentation




### variable Kind 

```C++
EKind FTryllTextPreview::Kind;
```




<hr>



### variable Message 

```C++
FString FTryllTextPreview::Message;
```




<hr>



### variable MissingLocales 

```C++
TArray<FString> FTryllTextPreview::MissingLocales;
```




<hr>



### variable Text 

```C++
FString FTryllTextPreview::Text;
```




<hr>



### variable bIsFallback 

```C++
bool FTryllTextPreview::bIsFallback;
```




<hr>
## Public Static Functions Documentation




### function Evaluate 

```C++
static FTryllTextPreview FTryllTextPreview::Evaluate (
    const UTryllTextProvider * Provider,
    FName Table,
    const FString & Id,
    const FString & PreviewLocale
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllTextProvider.h`

