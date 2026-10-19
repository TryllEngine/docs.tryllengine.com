

# Struct FTryllGameMemorySnapshot



[**ClassList**](annotated.md) **>** [**FTryllGameMemorySnapshot**](struct_f_tryll_game_memory_snapshot.md)


























## Public Attributes

| Type | Name |
| ---: | :--- |
|  uint32 | [**FormatVersion**](#variable-formatversion)   = `TryllGameMemoryFormatVersion`<br> |
|  FString | [**SaveId**](#variable-saveid)  <br> |
|  uint64 | [**SavedAtMs**](#variable-savedatms)   = `0`<br> |
|  TArray&lt; [**FTryllMutableKbSnapshot**](struct_f_tryll_mutable_kb_snapshot.md) &gt; | [**Stores**](#variable-stores)  <br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  bool | [**ToFlatBufferBytes**](#function-toflatbufferbytes) (TArray&lt; uint8 &gt; & OutBytes, [**FTryllError**](struct_f_tryll_error.md) & OutError) const<br> |


## Public Static Functions

| Type | Name |
| ---: | :--- |
|  bool | [**FromFlatBufferBytes**](#function-fromflatbufferbytes) (TArrayView&lt; const uint8 &gt; Bytes, [**FTryllGameMemorySnapshot**](struct_f_tryll_game_memory_snapshot.md) & OutSnapshot, [**FTryllError**](struct_f_tryll_error.md) & OutError) <br> |


























## Public Attributes Documentation




### variable FormatVersion 

```C++
uint32 FTryllGameMemorySnapshot::FormatVersion;
```




<hr>



### variable SaveId 

```C++
FString FTryllGameMemorySnapshot::SaveId;
```




<hr>



### variable SavedAtMs 

```C++
uint64 FTryllGameMemorySnapshot::SavedAtMs;
```




<hr>



### variable Stores 

```C++
TArray<FTryllMutableKbSnapshot> FTryllGameMemorySnapshot::Stores;
```




<hr>
## Public Functions Documentation




### function ToFlatBufferBytes 

```C++
bool FTryllGameMemorySnapshot::ToFlatBufferBytes (
    TArray< uint8 > & OutBytes,
    FTryllError & OutError
) const
```




<hr>
## Public Static Functions Documentation




### function FromFlatBufferBytes 

```C++
static bool FTryllGameMemorySnapshot::FromFlatBufferBytes (
    TArrayView< const uint8 > Bytes,
    FTryllGameMemorySnapshot & OutSnapshot,
    FTryllError & OutError
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllMkbSnapshot.h`

