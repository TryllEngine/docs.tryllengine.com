

# File TryllDialogSnapshot.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllDialogSnapshot.h**](_tryll_dialog_snapshot_8h.md)





* `#include "CoreMinimal.h"`
* `#include "TryllError.h"`
* `#include "Misc/Optional.h"`
* `#include "Misc/TVariant.h"`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTryllDialogImportResult**](struct_f_tryll_dialog_import_result.md) <br> |
| struct | [**FTryllDialogIntention**](struct_f_tryll_dialog_intention.md) <br> |
| struct | [**FTryllDialogInteractionSnapshot**](struct_f_tryll_dialog_interaction_snapshot.md) <br> |
| struct | [**FTryllDialogKnowledge**](struct_f_tryll_dialog_knowledge.md) <br> |
| struct | [**FTryllDialogKnowledgeChunk**](struct_f_tryll_dialog_knowledge_chunk.md) <br> |
| struct | [**FTryllDialogProjectionWindow**](struct_f_tryll_dialog_projection_window.md) <br> |
| struct | [**FTryllDialogSlot**](struct_f_tryll_dialog_slot.md) <br> |
| struct | [**FTryllDialogSnapshot**](struct_f_tryll_dialog_snapshot.md) <br> |
| struct | [**FTryllDialogSummary**](struct_f_tryll_dialog_summary.md) <br> |
| struct | [**FTryllDialogToolCallSnapshot**](struct_f_tryll_dialog_tool_call_snapshot.md) <br> |
| struct | [**FTryllDialogToolResult**](struct_f_tryll_dialog_tool_result.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| enum uint8 | [**ETryllDialogCompatibility**](#enum-etrylldialogcompatibility)  <br> |
| enum uint8 | [**ETryllDialogSlotKind**](#enum-etrylldialogslotkind)  <br> |
| typedef TVariant&lt; [**FTryllDialogSlot**](struct_f_tryll_dialog_slot.md), [**FTryllDialogKnowledge**](struct_f_tryll_dialog_knowledge.md), [**FTryllDialogIntention**](struct_f_tryll_dialog_intention.md), [**FTryllDialogToolCallSnapshot**](struct_f_tryll_dialog_tool_call_snapshot.md), [**FTryllDialogToolResult**](struct_f_tryll_dialog_tool_result.md), [**FTryllDialogSummary**](struct_f_tryll_dialog_summary.md) &gt; | [**FTryllDialogComponent**](#typedef-ftrylldialogcomponent)  <br> |
















































## Public Types Documentation




### enum ETryllDialogCompatibility 

```C++
enum ETryllDialogCompatibility {
    Strict = 0,
    AllowGraphMismatch = 1
};
```




<hr>



### enum ETryllDialogSlotKind 

```C++
enum ETryllDialogSlotKind {
    User = 0,
    Instruction = 1,
    General = 2
};
```




<hr>



### typedef FTryllDialogComponent 

```C++
using FTryllDialogComponent = TVariant<
   FTryllDialogSlot,
   FTryllDialogKnowledge,
   FTryllDialogIntention,
   FTryllDialogToolCallSnapshot,
   FTryllDialogToolResult,
   FTryllDialogSummary>;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllDialogSnapshot.h`

