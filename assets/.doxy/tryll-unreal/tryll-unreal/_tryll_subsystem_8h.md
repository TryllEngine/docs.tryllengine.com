

# File TryllSubsystem.h



[**FileList**](files.md) **>** [**client-unreal**](dir_95d666eee9112f2bfa7f2b736b6243b9.md) **>** [**Source**](dir_e37f8a870dc803113e92bf135247a735.md) **>** [**TryllClient**](dir_0869abba98a308e3c3eadd7e169e0f62.md) **>** [**Public**](dir_338741d27b4bda5805009de80ddaf6fc.md) **>** [**TryllSubsystem.h**](_tryll_subsystem_8h.md)





* `#include "CoreMinimal.h"`
* `#include "Containers/Ticker.h"`
* `#include "Subsystems/GameInstanceSubsystem.h"`
* `#include "Tickable.h"`
* `#include "TryllError.h"`
* `#include "TryllGraphDescription.h"`
* `#include "TryllModelInfo.h"`
* `#include "TryllVoiceInput.h"`
* `#include "TryllSubsystem.generated.h"`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTryllNodeEventKeyValue**](struct_f_tryll_node_event_key_value.md) <br> |
| class | [**UTryllSubsystem**](class_u_tryll_subsystem.md) <br> |
| struct | [**FEmbeddedStorageInfo**](struct_u_tryll_subsystem_1_1_f_embedded_storage_info.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| typedef void(TSharedPtr&lt; [**FTryllAgent**](class_f_tryll_agent.md) &gt;, [**FTryllError**](struct_f_tryll_error.md)) | [**FTryllOnAgentCreated**](#typedef-ftryllonagentcreated)  <br> |
| typedef void([**FTryllError**](struct_f_tryll_error.md)) | [**FTryllOnConfigureSession**](#typedef-ftryllonconfiguresession)  <br> |
| typedef void(TArray&lt; [**FTryllModelInfo**](struct_f_tryll_model_info.md) &gt;, [**FTryllError**](struct_f_tryll_error.md)) | [**FTryllOnListModels**](#typedef-ftryllonlistmodels)  <br> |
| typedef void(TSharedPtr&lt; [**FTryllVoiceInput**](class_f_tryll_voice_input.md) &gt;, [**FTryllError**](struct_f_tryll_error.md)) | [**FTryllOnVoiceInputCreated**](#typedef-ftryllonvoiceinputcreated)  <br> |




















## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_FiveParams**](#function-declare_dynamic_multicast_delegate_fiveparams) (FOnTryllIntentClassified, int64, AgentId, const FString &, Intent, const FString &, RecordId, int64, RecordIndex, float, Distance) <br>_Intent classification typed event — fired for NodeEvent event\_type="intent\_classified"._  |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_FourParams**](#function-declare_dynamic_multicast_delegate_fourparams) (FOnTryllNodeEvent, int64, AgentId, const FString &, NodeName, const FString &, EventType, const TArray&lt; [**FTryllNodeEventKeyValue**](struct_f_tryll_node_event_key_value.md) &gt; &, KvPairs) <br> |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_OneParam**](#function-declare_dynamic_multicast_delegate_oneparam) (FOnTryllConnectionChanged, bool, bConnected) <br> |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_OneParam**](#function-declare_dynamic_multicast_delegate_oneparam) (FOnTryllError, const FString &, ErrorMessage) <br> |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_OneParam**](#function-declare_dynamic_multicast_delegate_oneparam) (FOnTryllAgentDestroyed, int64, AgentId) <br> |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_OneParam**](#function-declare_dynamic_multicast_delegate_oneparam) (FOnTryllConfigureSession, bool, bSuccess) <br> |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_ThreeParams**](#function-declare_dynamic_multicast_delegate_threeparams) (FOnTryllToolCall, int64, AgentId, const FString &, ToolName, const FString &, ArgumentsJson) <br> |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_dynamic_multicast_delegate_twoparams) (FOnTryllListModels, const TArray&lt; [**FTryllModelInfo**](struct_f_tryll_model_info.md) &gt; &, Models, bool, bSuccess) <br> |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_dynamic_multicast_delegate_twoparams) (FOnTryllDownloadProgress, const FString &, ModelName, float, Percent) <br> |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_dynamic_multicast_delegate_twoparams) (FOnTryllDownloadComplete, const FString &, ModelName, bool, bSuccess) <br> |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_dynamic_multicast_delegate_twoparams) (FOnTryllLoadModel, const FString &, ModelName, bool, bSuccess) <br> |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_dynamic_multicast_delegate_twoparams) (FOnTryllUnloadModel, const FString &, ModelName, bool, bSuccess) <br> |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_dynamic_multicast_delegate_twoparams) (FOnTryllModelReady, const FString &, ModelName, bool, bSuccess) <br> |




























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
## Public Functions Documentation




### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_FiveParams 

_Intent classification typed event — fired for NodeEvent event\_type="intent\_classified"._ 
```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FiveParams (
    FOnTryllIntentClassified,
    int64,
    AgentId,
    const FString &,
    Intent,
    const FString &,
    RecordId,
    int64,
    RecordIndex,
    float,
    Distance
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_FourParams 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FourParams (
    FOnTryllNodeEvent,
    int64,
    AgentId,
    const FString &,
    NodeName,
    const FString &,
    EventType,
    const TArray< FTryllNodeEventKeyValue > &,
    KvPairs
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_OneParam 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam (
    FOnTryllConnectionChanged,
    bool,
    bConnected
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_OneParam 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam (
    FOnTryllError,
    const FString &,
    ErrorMessage
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_OneParam 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam (
    FOnTryllAgentDestroyed,
    int64,
    AgentId
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_OneParam 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam (
    FOnTryllConfigureSession,
    bool,
    bSuccess
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_ThreeParams 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParams (
    FOnTryllToolCall,
    int64,
    AgentId,
    const FString &,
    ToolName,
    const FString &,
    ArgumentsJson
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams (
    FOnTryllListModels,
    const TArray< FTryllModelInfo > &,
    Models,
    bool,
    bSuccess
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams (
    FOnTryllDownloadProgress,
    const FString &,
    ModelName,
    float,
    Percent
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams (
    FOnTryllDownloadComplete,
    const FString &,
    ModelName,
    bool,
    bSuccess
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams (
    FOnTryllLoadModel,
    const FString &,
    ModelName,
    bool,
    bSuccess
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams (
    FOnTryllUnloadModel,
    const FString &,
    ModelName,
    bool,
    bSuccess
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams (
    FOnTryllModelReady,
    const FString &,
    ModelName,
    bool,
    bSuccess
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/TryllSubsystem.h`

