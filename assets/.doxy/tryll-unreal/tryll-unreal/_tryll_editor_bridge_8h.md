

# File TryllEditorBridge.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllEditorBridge.h**](_tryll_editor_bridge_8h.md)





* `#include "CoreMinimal.h"`
* `#include "TryllGraphDescription.h"`
* `#include "TryllKvCache.h"`
* `#include "TryllModelInfo.h"`
* `#include "TryllRuntimeSettings.h"`
* `#include "TryllToolCall.h"`
* `#include "TryllVariableDecl.h"`
* `#include "TryllVoiceInput.h"`
* `#include "TryllSubsystem.h"`















## Classes

| Type | Name |
| ---: | :--- |
| class | [**FTryllEditorBridge**](class_f_tryll_editor_bridge.md) <br> |
| class | [**ITryllEditorBridge**](class_i_tryll_editor_bridge.md) <br> |






















## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_FiveParams**](#function-declare_multicast_delegate_fiveparams) (FTryllOnIntentClassifiedNative, uint64, const FString &, const FString &, int64, float) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_FourParams**](#function-declare_multicast_delegate_fourparams) (FTryllOnDownloadProgressNative, const FString &, uint64, uint64, float) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_FourParams**](#function-declare_multicast_delegate_fourparams) (FTryllOnToolCallWithIdNative, uint64, const FString &, const FString &, const FString &) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_FourParams**](#function-declare_multicast_delegate_fourparams) (FTryllOnNodeEventNative, uint64, const FString &, const FString &, const TArray&lt; [**FTryllNodeEventKeyValue**](struct_f_tryll_node_event_key_value.md) &gt; &) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_OneParam**](#function-declare_multicast_delegate_oneparam) (FTryllOnConnectionChangedNative, bool) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_OneParam**](#function-declare_multicast_delegate_oneparam) (FTryllOnErrorNative, const FString &) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_OneParam**](#function-declare_multicast_delegate_oneparam) (FTryllOnAgentDestroyedNative, uint64) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_OneParam**](#function-declare_multicast_delegate_oneparam) (FTryllOnCreateSessionCompleteNative, bool) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_ThreeParams**](#function-declare_multicast_delegate_threeparams) (FTryllOnDownloadCompleteNative, const FString &, bool, const FString &) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_ThreeParams**](#function-declare_multicast_delegate_threeparams) (FTryllOnCreateEmbeddedStringStorageCompleteNative, const FString &, int32, bool) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_ThreeParams**](#function-declare_multicast_delegate_threeparams) (FTryllOnToolCallNative, uint64, const FString &, const FString &) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_ThreeParams**](#function-declare_multicast_delegate_threeparams) (FTryllOnPausedNative, uint64, const FString &, const FString &) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_multicast_delegate_twoparams) (FTryllOnListModelsCompleteNative, const TArray&lt; [**FTryllModelInfo**](struct_f_tryll_model_info.md) &gt; &, bool) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_multicast_delegate_twoparams) (FTryllOnLoadModelCompleteNative, const FString &, bool) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_multicast_delegate_twoparams) (FTryllOnUnloadModelCompleteNative, const FString &, bool) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_multicast_delegate_twoparams) (FTryllOnDeleteModelCompleteNative, const FString &, bool) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_multicast_delegate_twoparams) (FTryllOnCreateStringStorageCompleteNative, const FString &, bool) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_multicast_delegate_twoparams) (FTryllOnDestroyStringStorageCompleteNative, const FString &, bool) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_multicast_delegate_twoparams) (FTryllOnDestroyEmbeddedStringStorageCompleteNative, const FString &, bool) <br> |
|   | [**DECLARE\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_multicast_delegate_twoparams) (FTryllOnToolCallDetailedNative, uint64, const [**FTryllToolCall**](struct_f_tryll_tool_call.md) &) <br> |




























## Public Functions Documentation




### function DECLARE\_MULTICAST\_DELEGATE\_FiveParams 

```C++
DECLARE_MULTICAST_DELEGATE_FiveParams (
    FTryllOnIntentClassifiedNative,
    uint64,
    const FString &,
    const FString &,
    int64,
    float
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_FourParams 

```C++
DECLARE_MULTICAST_DELEGATE_FourParams (
    FTryllOnDownloadProgressNative,
    const FString &,
    uint64,
    uint64,
    float
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_FourParams 

```C++
DECLARE_MULTICAST_DELEGATE_FourParams (
    FTryllOnToolCallWithIdNative,
    uint64,
    const FString &,
    const FString &,
    const FString &
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_FourParams 

```C++
DECLARE_MULTICAST_DELEGATE_FourParams (
    FTryllOnNodeEventNative,
    uint64,
    const FString &,
    const FString &,
    const TArray< FTryllNodeEventKeyValue > &
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_OneParam 

```C++
DECLARE_MULTICAST_DELEGATE_OneParam (
    FTryllOnConnectionChangedNative,
    bool
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_OneParam 

```C++
DECLARE_MULTICAST_DELEGATE_OneParam (
    FTryllOnErrorNative,
    const FString &
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_OneParam 

```C++
DECLARE_MULTICAST_DELEGATE_OneParam (
    FTryllOnAgentDestroyedNative,
    uint64
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_OneParam 

```C++
DECLARE_MULTICAST_DELEGATE_OneParam (
    FTryllOnCreateSessionCompleteNative,
    bool
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_ThreeParams 

```C++
DECLARE_MULTICAST_DELEGATE_ThreeParams (
    FTryllOnDownloadCompleteNative,
    const FString &,
    bool,
    const FString &
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_ThreeParams 

```C++
DECLARE_MULTICAST_DELEGATE_ThreeParams (
    FTryllOnCreateEmbeddedStringStorageCompleteNative,
    const FString &,
    int32,
    bool
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_ThreeParams 

```C++
DECLARE_MULTICAST_DELEGATE_ThreeParams (
    FTryllOnToolCallNative,
    uint64,
    const FString &,
    const FString &
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_ThreeParams 

```C++
DECLARE_MULTICAST_DELEGATE_ThreeParams (
    FTryllOnPausedNative,
    uint64,
    const FString &,
    const FString &
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_MULTICAST_DELEGATE_TwoParams (
    FTryllOnListModelsCompleteNative,
    const TArray< FTryllModelInfo > &,
    bool
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_MULTICAST_DELEGATE_TwoParams (
    FTryllOnLoadModelCompleteNative,
    const FString &,
    bool
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_MULTICAST_DELEGATE_TwoParams (
    FTryllOnUnloadModelCompleteNative,
    const FString &,
    bool
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_MULTICAST_DELEGATE_TwoParams (
    FTryllOnDeleteModelCompleteNative,
    const FString &,
    bool
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_MULTICAST_DELEGATE_TwoParams (
    FTryllOnCreateStringStorageCompleteNative,
    const FString &,
    bool
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_MULTICAST_DELEGATE_TwoParams (
    FTryllOnDestroyStringStorageCompleteNative,
    const FString &,
    bool
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_MULTICAST_DELEGATE_TwoParams (
    FTryllOnDestroyEmbeddedStringStorageCompleteNative,
    const FString &,
    bool
) 
```




<hr>



### function DECLARE\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_MULTICAST_DELEGATE_TwoParams (
    FTryllOnToolCallDetailedNative,
    uint64,
    const FTryllToolCall &
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllEditorBridge.h`

