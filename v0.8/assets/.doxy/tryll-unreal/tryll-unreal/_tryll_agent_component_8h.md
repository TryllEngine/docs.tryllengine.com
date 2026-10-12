

# File TryllAgentComponent.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllAgentComponent.h**](_tryll_agent_component_8h.md)





* `#include "CoreMinimal.h"`
* `#include "Components/ActorComponent.h"`
* `#include "TryllGraphDescription.h"`
* `#include "TryllError.h"`
* `#include "TryllKvCache.h"`
* `#include "TryllThrottle.h"`
* `#include "TryllDialogInteraction.h"`
* `#include "TryllDialogSnapshot.h"`
* `#include "TryllVariableDecl.h"`
* `#include "TryllToolCall.h"`
* `#include "TryllIntentClassification.h"`
* `#include "TryllNodeEvent.h"`
* `#include "Generated/Nodes/TryllNodeParamsBase.h"`
* `#include "TryllAgentComponent.generated.h"`















## Classes

| Type | Name |
| ---: | :--- |
| class | [**UTryllAgentComponent**](class_u_tryll_agent_component.md) <br> |






















## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_multicast_delegate_twoparams) (FOnTryllDialogExportedNative, const [**FTryllDialogSnapshot**](struct_f_tryll_dialog_snapshot.md) &, const [**FTryllError**](struct_f_tryll_error.md) &) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_multicast_delegate_twoparams) (FOnTryllDialogImportedNative, const [**FTryllDialogImportResult**](struct_f_tryll_dialog_import_result.md) &, const [**FTryllError**](struct_f_tryll_error.md) &) <br> |




























## Public Functions Documentation




### function DECLARE\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_MULTICAST_DELEGATE_TwoParams (
    FOnTryllDialogExportedNative,
    const FTryllDialogSnapshot &,
    const FTryllError &
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_MULTICAST_DELEGATE_TwoParams (
    FOnTryllDialogImportedNative,
    const FTryllDialogImportResult &,
    const FTryllError &
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllAgentComponent.h`

