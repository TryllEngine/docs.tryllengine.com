

# File TryllSubsystem.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllSubsystem.h**](_tryll_subsystem_8h.md)





* `#include "CoreMinimal.h"`
* `#include "Containers/Ticker.h"`
* `#include "Subsystems/GameInstanceSubsystem.h"`
* `#include "Tickable.h"`
* `#include "TryllError.h"`
* `#include "TryllGraphDescription.h"`
* `#include "TryllModelInfo.h"`
* `#include "TryllVoiceInput.h"`
* `#include "TryllRuntimeSettings.h"`
* `#include "TryllSubsystem.generated.h"`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTryllNodeEventKeyValue**](struct_f_tryll_node_event_key_value.md) <br>_Intent classification typed event — fired for NodeEvent event\_type="intent\_classified"._  |
| class | [**UTryllSubsystem**](class_u_tryll_subsystem.md) <br> |
| struct | [**FEmbeddedStorageInfo**](struct_u_tryll_subsystem_1_1_f_embedded_storage_info.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| typedef void(TSharedPtr&lt; [**FTryllAgent**](class_f_tryll_agent.md) &gt;, [**FTryllError**](struct_f_tryll_error.md)) | [**FTryllOnAgentCreated**](#typedef-ftryllonagentcreated)  <br> |
| typedef void([**FTryllError**](struct_f_tryll_error.md)) | [**FTryllOnConfigureSession**](#typedef-ftryllonconfiguresession)  <br> |
| typedef void(TArray&lt; [**FTryllModelInfo**](struct_f_tryll_model_info.md) &gt;, [**FTryllError**](struct_f_tryll_error.md)) | [**FTryllOnListModels**](#typedef-ftryllonlistmodels)  <br> |
| typedef void(TSharedPtr&lt; [**FTryllVoiceInput**](class_f_tryll_voice_input.md) &gt;, [**FTryllError**](struct_f_tryll_error.md)) | [**FTryllOnVoiceInputCreated**](#typedef-ftryllonvoiceinputcreated)  <br> |
















































## Public Types Documentation




### typedef FTryllOnAgentCreated 

```C++
using FTryllOnAgentCreated = void(TSharedPtr<FTryllAgent>, FTryllError);
```




<hr>



### typedef FTryllOnConfigureSession 

```C++
using FTryllOnConfigureSession = void(FTryllError);
```




<hr>



### typedef FTryllOnListModels 

```C++
using FTryllOnListModels = void(TArray<FTryllModelInfo>, FTryllError);
```




<hr>



### typedef FTryllOnVoiceInputCreated 

```C++
using FTryllOnVoiceInputCreated = void(TSharedPtr<FTryllVoiceInput>, FTryllError);
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllSubsystem.h`

