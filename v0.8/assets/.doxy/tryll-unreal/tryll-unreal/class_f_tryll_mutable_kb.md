

# Class FTryllMutableKb



[**ClassList**](annotated.md) **>** [**FTryllMutableKb**](class_f_tryll_mutable_kb.md)



[More...](#detailed-description)

* `#include <TryllMutableKb.h>`



Inherits the following classes: TSharedFromThis< FTryllMutableKb >


































## Public Functions

| Type | Name |
| ---: | :--- |
|  void | [**Clear**](#function-clear) (TFunction&lt; void(uint32 ClearedCount, const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete) <br> |
|  void | [**Delete**](#function-delete) (const TArray&lt; FString &gt; & Ids, TFunction&lt; void(uint32 DeletedCount, const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete) <br> |
|  void | [**DrainPendingCallbacks**](#function-drainpendingcallbacks) (const [**FTryllError**](struct_f_tryll_error.md) & TerminalError) <br> |
|  void | [**Export**](#function-export) (TFunction&lt; void(const [**FTryllMutableKbSnapshot**](struct_f_tryll_mutable_kb_snapshot.md) &, const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete) <br> |
|   | [**FTryllMutableKb**](#function-ftryllmutablekb) (TSharedPtr&lt; FTryllConnection &gt; InConnection, FString InName, FString InSchema) <br> |
|  void | [**Get**](#function-get) (const FString & Id, TFunction&lt; void(bool bFound, const [**FTryllMutableKbRecord**](struct_f_tryll_mutable_kb_record.md) &, const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete) <br> |
|  const FString & | [**GetName**](#function-getname) () const<br> |
|  const FString & | [**GetSchema**](#function-getschema) () const<br> |
|  void | [**Import**](#function-import) (const [**FTryllMutableKbSnapshot**](struct_f_tryll_mutable_kb_snapshot.md) & Snapshot, ETryllMutableKbCompatibility Compatibility, TFunction&lt; void(const [**FTryllMutableKbImportResult**](struct_f_tryll_mutable_kb_import_result.md) &, const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete) <br> |
|  bool | [**IsValid**](#function-isvalid) () const<br> |
|  void | [**List**](#function-list) (uint32 Offset, uint32 Limit, TFunction&lt; void(const [**FTryllMutableKbListPage**](struct_f_tryll_mutable_kb_list_page.md) &, const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete) <br> |
|  void | [**Query**](#function-query-12) (const FString & QueryText, uint32 TopK, TFunction&lt; void(const TArray&lt; [**FTryllMutableKbHit**](struct_f_tryll_mutable_kb_hit.md) &gt; &, const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete) <br> |
|  void | [**Query**](#function-query-22) (const FString & QueryText, uint32 TopK, ETryllRetrievalMode Mode, TFunction&lt; void(const TArray&lt; [**FTryllMutableKbHit**](struct_f_tryll_mutable_kb_hit.md) &gt; &, const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete) <br> |
|  void | [**Stats**](#function-stats) (TFunction&lt; void(const [**FTryllMutableKbStats**](struct_f_tryll_mutable_kb_stats.md) &, const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete) <br> |
|  bool | [**TryHandleEvent**](#function-tryhandleevent) (const struct FTryllConnEvent & Event) <br> |
|  void | [**Write**](#function-write) (const TArray&lt; [**FTryllMutableKbUpsert**](struct_f_tryll_mutable_kb_upsert.md) &gt; & Records, TFunction&lt; void(const TArray&lt; [**FTryllMutableKbWriteResult**](struct_f_tryll_mutable_kb_write_result.md) &gt; &, const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete) <br> |




























## Detailed Description


Session-bound handle for one named mutable knowledge base. Created by [**UTryllSubsystem**](class_u_tryll_subsystem.md) / UTryllEditorSubsystem after EnsureMutableKb. Operations fail with a terminal error after the owning session is torn down. 


    
## Public Functions Documentation




### function Clear 

```C++
void FTryllMutableKb::Clear (
    TFunction< void(uint32 ClearedCount, const FTryllError &)> OnComplete
) 
```




<hr>



### function Delete 

```C++
void FTryllMutableKb::Delete (
    const TArray< FString > & Ids,
    TFunction< void(uint32 DeletedCount, const FTryllError &)> OnComplete
) 
```




<hr>



### function DrainPendingCallbacks 

```C++
void FTryllMutableKb::DrainPendingCallbacks (
    const FTryllError & TerminalError
) 
```




<hr>



### function Export 

```C++
void FTryllMutableKb::Export (
    TFunction< void(const FTryllMutableKbSnapshot &, const FTryllError &)> OnComplete
) 
```




<hr>



### function FTryllMutableKb 

```C++
FTryllMutableKb::FTryllMutableKb (
    TSharedPtr< FTryllConnection > InConnection,
    FString InName,
    FString InSchema
) 
```




<hr>



### function Get 

```C++
void FTryllMutableKb::Get (
    const FString & Id,
    TFunction< void(bool bFound, const FTryllMutableKbRecord &, const FTryllError &)> OnComplete
) 
```




<hr>



### function GetName 

```C++
inline const FString & FTryllMutableKb::GetName () const
```




<hr>



### function GetSchema 

```C++
inline const FString & FTryllMutableKb::GetSchema () const
```




<hr>



### function Import 

```C++
void FTryllMutableKb::Import (
    const FTryllMutableKbSnapshot & Snapshot,
    ETryllMutableKbCompatibility Compatibility,
    TFunction< void(const FTryllMutableKbImportResult &, const FTryllError &)> OnComplete
) 
```




<hr>



### function IsValid 

```C++
bool FTryllMutableKb::IsValid () const
```




<hr>



### function List 

```C++
void FTryllMutableKb::List (
    uint32 Offset,
    uint32 Limit,
    TFunction< void(const FTryllMutableKbListPage &, const FTryllError &)> OnComplete
) 
```




<hr>



### function Query [1/2]

```C++
void FTryllMutableKb::Query (
    const FString & QueryText,
    uint32 TopK,
    TFunction< void(const TArray< FTryllMutableKbHit > &, const FTryllError &)> OnComplete
) 
```




<hr>



### function Query [2/2]

```C++
void FTryllMutableKb::Query (
    const FString & QueryText,
    uint32 TopK,
    ETryllRetrievalMode Mode,
    TFunction< void(const TArray< FTryllMutableKbHit > &, const FTryllError &)> OnComplete
) 
```




<hr>



### function Stats 

```C++
void FTryllMutableKb::Stats (
    TFunction< void(const FTryllMutableKbStats &, const FTryllError &)> OnComplete
) 
```




<hr>



### function TryHandleEvent 

```C++
bool FTryllMutableKb::TryHandleEvent (
    const struct FTryllConnEvent & Event
) 
```




<hr>



### function Write 

```C++
void FTryllMutableKb::Write (
    const TArray< FTryllMutableKbUpsert > & Records,
    TFunction< void(const TArray< FTryllMutableKbWriteResult > &, const FTryllError &)> OnComplete
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllMutableKb.h`

