

# Struct FTryllMutableKbSnapshot



[**ClassList**](annotated.md) **>** [**FTryllMutableKbSnapshot**](struct_f_tryll_mutable_kb_snapshot.md)


























## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**EmbeddingModel**](#variable-embeddingmodel)  <br> |
|  FString | [**Name**](#variable-name)  <br> |
|  uint64 | [**NextGeneratedId**](#variable-nextgeneratedid)   = `1`<br> |
|  TArray&lt; [**FTryllMutableKbRecord**](struct_f_tryll_mutable_kb_record.md) &gt; | [**Records**](#variable-records)  <br> |
|  FString | [**SchemaFingerprint**](#variable-schemafingerprint)  <br> |
|  FString | [**SchemaName**](#variable-schemaname)  <br> |
|  uint64 | [**WriteGeneration**](#variable-writegeneration)   = `0`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  bool | [**ToFlatBufferBytes**](#function-toflatbufferbytes) (TArray&lt; uint8 &gt; & OutBytes, [**FTryllError**](struct_f_tryll_error.md) & OutError) const<br> |


## Public Static Functions

| Type | Name |
| ---: | :--- |
|  bool | [**FromFlatBufferBytes**](#function-fromflatbufferbytes) (TArrayView&lt; const uint8 &gt; Bytes, [**FTryllMutableKbSnapshot**](struct_f_tryll_mutable_kb_snapshot.md) & OutSnapshot, [**FTryllError**](struct_f_tryll_error.md) & OutError) <br> |


























## Public Attributes Documentation




### variable EmbeddingModel 

```C++
FString FTryllMutableKbSnapshot::EmbeddingModel;
```




<hr>



### variable Name 

```C++
FString FTryllMutableKbSnapshot::Name;
```




<hr>



### variable NextGeneratedId 

```C++
uint64 FTryllMutableKbSnapshot::NextGeneratedId;
```




<hr>



### variable Records 

```C++
TArray<FTryllMutableKbRecord> FTryllMutableKbSnapshot::Records;
```




<hr>



### variable SchemaFingerprint 

```C++
FString FTryllMutableKbSnapshot::SchemaFingerprint;
```




<hr>



### variable SchemaName 

```C++
FString FTryllMutableKbSnapshot::SchemaName;
```




<hr>



### variable WriteGeneration 

```C++
uint64 FTryllMutableKbSnapshot::WriteGeneration;
```




<hr>
## Public Functions Documentation




### function ToFlatBufferBytes 

```C++
bool FTryllMutableKbSnapshot::ToFlatBufferBytes (
    TArray< uint8 > & OutBytes,
    FTryllError & OutError
) const
```




<hr>
## Public Static Functions Documentation




### function FromFlatBufferBytes 

```C++
static bool FTryllMutableKbSnapshot::FromFlatBufferBytes (
    TArrayView< const uint8 > Bytes,
    FTryllMutableKbSnapshot & OutSnapshot,
    FTryllError & OutError
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllMkbSnapshot.h`

