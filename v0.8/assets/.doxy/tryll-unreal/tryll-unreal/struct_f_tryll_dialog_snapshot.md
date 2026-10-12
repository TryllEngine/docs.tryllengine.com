

# Struct FTryllDialogSnapshot



[**ClassList**](annotated.md) **>** [**FTryllDialogSnapshot**](struct_f_tryll_dialog_snapshot.md)


























## Public Attributes

| Type | Name |
| ---: | :--- |
|  uint32 | [**FormatVersion**](#variable-formatversion)   = `1`<br> |
|  FString | [**GraphSignature**](#variable-graphsignature)  <br> |
|  TArray&lt; [**FTryllDialogInteractionSnapshot**](struct_f_tryll_dialog_interaction_snapshot.md) &gt; | [**Interactions**](#variable-interactions)  <br> |
|  uint64 | [**NextExchangeId**](#variable-nextexchangeid)   = `1`<br> |
|  uint64 | [**NextToolCallSeq**](#variable-nexttoolcallseq)   = `0`<br> |
|  TArray&lt; [**FTryllDialogProjectionWindow**](struct_f_tryll_dialog_projection_window.md) &gt; | [**ProjectionWindows**](#variable-projectionwindows)  <br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  bool | [**ToFlatBufferBytes**](#function-toflatbufferbytes) (TArray&lt; uint8 &gt; & OutBytes, [**FTryllError**](struct_f_tryll_error.md) & OutError) const<br> |


## Public Static Functions

| Type | Name |
| ---: | :--- |
|  bool | [**FromFlatBufferBytes**](#function-fromflatbufferbytes) (TArrayView&lt; const uint8 &gt; Bytes, [**FTryllDialogSnapshot**](struct_f_tryll_dialog_snapshot.md) & OutSnapshot, [**FTryllError**](struct_f_tryll_error.md) & OutError) <br> |


























## Public Attributes Documentation




### variable FormatVersion 

```C++
uint32 FTryllDialogSnapshot::FormatVersion;
```




<hr>



### variable GraphSignature 

```C++
FString FTryllDialogSnapshot::GraphSignature;
```




<hr>



### variable Interactions 

```C++
TArray<FTryllDialogInteractionSnapshot> FTryllDialogSnapshot::Interactions;
```




<hr>



### variable NextExchangeId 

```C++
uint64 FTryllDialogSnapshot::NextExchangeId;
```




<hr>



### variable NextToolCallSeq 

```C++
uint64 FTryllDialogSnapshot::NextToolCallSeq;
```




<hr>



### variable ProjectionWindows 

```C++
TArray<FTryllDialogProjectionWindow> FTryllDialogSnapshot::ProjectionWindows;
```




<hr>
## Public Functions Documentation




### function ToFlatBufferBytes 

```C++
bool FTryllDialogSnapshot::ToFlatBufferBytes (
    TArray< uint8 > & OutBytes,
    FTryllError & OutError
) const
```




<hr>
## Public Static Functions Documentation




### function FromFlatBufferBytes 

```C++
static bool FTryllDialogSnapshot::FromFlatBufferBytes (
    TArrayView< const uint8 > Bytes,
    FTryllDialogSnapshot & OutSnapshot,
    FTryllError & OutError
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllDialogSnapshot.h`

