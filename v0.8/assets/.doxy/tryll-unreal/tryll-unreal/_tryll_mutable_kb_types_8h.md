

# File TryllMutableKbTypes.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllMutableKbTypes.h**](_tryll_mutable_kb_types_8h.md)





* `#include "CoreMinimal.h"`
* `#include "TryllGraphDescription.h"`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTryllGameMemoryLoadResult**](struct_f_tryll_game_memory_load_result.md) <br> |
| struct | [**FTryllGameMemorySaveResult**](struct_f_tryll_game_memory_save_result.md) <br> |
| struct | [**FTryllMutableKbHit**](struct_f_tryll_mutable_kb_hit.md) <br> |
| struct | [**FTryllMutableKbImportResult**](struct_f_tryll_mutable_kb_import_result.md) <br> |
| struct | [**FTryllMutableKbInfo**](struct_f_tryll_mutable_kb_info.md) <br> |
| struct | [**FTryllMutableKbListPage**](struct_f_tryll_mutable_kb_list_page.md) <br> |
| struct | [**FTryllMutableKbRecord**](struct_f_tryll_mutable_kb_record.md) <br> |
| struct | [**FTryllMutableKbStats**](struct_f_tryll_mutable_kb_stats.md) <br> |
| struct | [**FTryllMutableKbUpsert**](struct_f_tryll_mutable_kb_upsert.md) <br> |
| struct | [**FTryllMutableKbWriteResult**](struct_f_tryll_mutable_kb_write_result.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| enum uint8 | [**ETryllMutableKbCompatibility**](#enum-etryllmutablekbcompatibility)  <br> |
| enum uint8 | [**ETryllMutableKbEmbedTiming**](#enum-etryllmutablekbembedtiming)  <br> |
















































## Public Types Documentation




### enum ETryllMutableKbCompatibility 

```C++
enum ETryllMutableKbCompatibility {
    Strict = 0,
    AllowSchemaDrift = 1
};
```




<hr>



### enum ETryllMutableKbEmbedTiming 

```C++
enum ETryllMutableKbEmbedTiming {
    Unspecified = 0,
    Now = 1,
    Lazy = 2
};
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllMutableKbTypes.h`

