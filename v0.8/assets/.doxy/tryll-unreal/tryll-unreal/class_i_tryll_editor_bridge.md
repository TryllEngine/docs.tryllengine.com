

# Class ITryllEditorBridge



[**ClassList**](annotated.md) **>** [**ITryllEditorBridge**](class_i_tryll_editor_bridge.md)



[More...](#detailed-description)

* `#include <TryllEditorBridge.h>`





































## Public Functions

| Type | Name |
| ---: | :--- |
| virtual void | [**AcquireConnection**](#function-acquireconnection) () = 0<br> |
| virtual void | [**AdoptRelaunchedServer**](#function-adoptrelaunchedserver) () = 0<br> |
| virtual void | [**CreateSession**](#function-createsession) (FString GameName=TEXT(""), FString Locale=TEXT("")) = 0<br> |
| virtual int32 | [**GetActualPort**](#function-getactualport) () const = 0<br> |
| virtual FTryllOnAgentDestroyedNative & | [**GetOnAgentDestroyedNative**](#function-getonagentdestroyednative) () = 0<br> |
| virtual FTryllOnConnectionChangedNative & | [**GetOnConnectionChangedNative**](#function-getonconnectionchangednative) () = 0<br> |
| virtual FTryllOnCreateEmbeddedStringStorageCompleteNative & | [**GetOnCreateEmbeddedStringStorageCompleteNative**](#function-getoncreateembeddedstringstoragecompletenative) () = 0<br> |
| virtual FTryllOnCreateSessionCompleteNative & | [**GetOnCreateSessionCompleteNative**](#function-getoncreatesessioncompletenative) () = 0<br> |
| virtual FTryllOnCreateStringStorageCompleteNative & | [**GetOnCreateStringStorageCompleteNative**](#function-getoncreatestringstoragecompletenative) () = 0<br> |
| virtual FTryllOnDestroyEmbeddedStringStorageCompleteNative & | [**GetOnDestroyEmbeddedStringStorageCompleteNative**](#function-getondestroyembeddedstringstoragecompletenative) () = 0<br> |
| virtual FTryllOnDestroyStringStorageCompleteNative & | [**GetOnDestroyStringStorageCompleteNative**](#function-getondestroystringstoragecompletenative) () = 0<br> |
| virtual FTryllOnDownloadCompleteNative & | [**GetOnDownloadCompleteNative**](#function-getondownloadcompletenative) () = 0<br> |
| virtual FTryllOnDownloadProgressNative & | [**GetOnDownloadProgressNative**](#function-getondownloadprogressnative) () = 0<br> |
| virtual FTryllOnErrorNative & | [**GetOnErrorNative**](#function-getonerrornative) () = 0<br> |
| virtual FTryllOnGetMemoryConsumptionCompleteNative & | [**GetOnGetMemoryConsumptionCompleteNative**](#function-getongetmemoryconsumptioncompletenative) () = 0<br> |
| virtual FTryllOnIntentClassifiedNative & | [**GetOnIntentClassifiedNative**](#function-getonintentclassifiednative) () = 0<br> |
| virtual FTryllOnIntentLlmClassifiedNative & | [**GetOnIntentLlmClassifiedNative**](#function-getonintentllmclassifiednative) () = 0<br> |
| virtual FTryllOnListModelsCompleteNative & | [**GetOnListModelsCompleteNative**](#function-getonlistmodelscompletenative) () = 0<br> |
| virtual FTryllOnLoadModelCompleteNative & | [**GetOnLoadModelCompleteNative**](#function-getonloadmodelcompletenative) () = 0<br> |
| virtual FTryllOnNodeEventNative & | [**GetOnNodeEventNative**](#function-getonnodeeventnative) () = 0<br> |
| virtual FTryllOnPausedForwardNative & | [**GetOnPausedForwardNative**](#function-getonpausedforwardnative) () = 0<br> |
| virtual FTryllOnPausedNative & | [**GetOnPausedNative**](#function-getonpausednative) () = 0<br> |
| virtual FTryllOnSetSessionLocaleCompleteNative & | [**GetOnSetSessionLocaleCompleteNative**](#function-getonsetsessionlocalecompletenative) () = 0<br> |
| virtual FTryllOnToolCallDetailedNative & | [**GetOnToolCallDetailedNative**](#function-getontoolcalldetailednative) () = 0<br> |
| virtual FTryllOnToolCallNative & | [**GetOnToolCallNative**](#function-getontoolcallnative) () = 0<br> |
| virtual FTryllOnToolCallWithIdNative & | [**GetOnToolCallWithIdNative**](#function-getontoolcallwithidnative) () = 0<br> |
| virtual FTryllOnUnloadModelCompleteNative & | [**GetOnUnloadModelCompleteNative**](#function-getonunloadmodelcompletenative) () = 0<br> |
| virtual int64 | [**GetSessionId**](#function-getsessionid) () const = 0<br> |
| virtual bool | [**HasSessionPauseListener**](#function-hassessionpauselistener) () const = 0<br>_Direct OnPausedNative listeners, or any registered probe that returns true._  |
| virtual bool | [**IsConnected**](#function-isconnected) () const = 0<br> |
| virtual bool | [**IsSessionReady**](#function-issessionready) () const = 0<br> |
| virtual int32 | [**RegisterPauseListenerProbe**](#function-registerpauselistenerprobe) (TFunction&lt; bool()&gt; Probe) = 0<br> |
| virtual void | [**ReleaseConnection**](#function-releaseconnection) () = 0<br> |
| virtual void | [**RequestCreateAgent**](#function-requestcreateagent) (const [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) & Graph, TFunction&lt; FTryllOnAgentCreated &gt; OnComplete, bool bEnableDiagnostics=false, bool bMaintainDialogueHistory=true, const TArray&lt; [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) &gt; & Variables={}, ETryllKvCacheInitialization KvCacheInitialization=ETryllKvCacheInitialization::AllocateOnly, ETryllWorkload Workload=ETryllWorkload::Interactive) = 0<br> |
| virtual void | [**RequestCreateEmbeddedStringStorage**](#function-requestcreateembeddedstringstorage) (const FString & Name, const FString & ConfigPath, const FString & EmbeddingModel=FString{}, TFunction&lt; UTryllSubsystem::FTryllOnEmbeddedStorageCreated &gt; OnComplete=nullptr) = 0<br> |
| virtual void | [**RequestCreateEmbeddedStringStorageFromStrings**](#function-requestcreateembeddedstringstoragefromstrings) (const FString & Name, const TArray&lt; FString &gt; & Strings, const FString & EmbeddingModel, TFunction&lt; UTryllSubsystem::FTryllOnEmbeddedStorageCreated &gt; OnComplete=nullptr) = 0<br> |
| virtual void | [**RequestCreateStringStorage**](#function-requestcreatestringstorage) (const FString & Name, const TArray&lt; FString &gt; & Strings, TFunction&lt; UTryllSubsystem::FTryllOnStringStorage &gt; OnComplete=nullptr) = 0<br> |
| virtual void | [**RequestCreateStringStorageFromFile**](#function-requestcreatestringstoragefromfile) (const FString & Name, const FString & FilePath, uint8 Kind=0, TFunction&lt; UTryllSubsystem::FTryllOnStringStorage &gt; OnComplete=nullptr) = 0<br> |
| virtual void | [**RequestCreateStringStorageKeyed**](#function-requestcreatestringstoragekeyed) (const FString & Name, const TArray&lt; FString &gt; & Keys, const TArray&lt; FString &gt; & Values, uint8 Kind=1, TFunction&lt; UTryllSubsystem::FTryllOnStringStorage &gt; OnComplete=nullptr) = 0<br> |
| virtual void | [**RequestCreateVoiceInput**](#function-requestcreatevoiceinput) (const [**FTryllVoiceInputConfig**](struct_f_tryll_voice_input_config.md) & Config, TFunction&lt; FTryllOnVoiceInputCreated &gt; OnComplete) = 0<br> |
| virtual void | [**RequestDestroyAgent**](#function-requestdestroyagent) (TSharedPtr&lt; [**FTryllAgent**](class_f_tryll_agent.md) &gt; Agent) = 0<br> |
| virtual void | [**RequestDestroyEmbeddedStringStorage**](#function-requestdestroyembeddedstringstorage) (const FString & Name, TFunction&lt; UTryllSubsystem::FTryllOnEmbeddedStorageDestroyed &gt; OnComplete=nullptr) = 0<br> |
| virtual void | [**RequestDestroyStringStorage**](#function-requestdestroystringstorage) (const FString & Name, TFunction&lt; UTryllSubsystem::FTryllOnStringStorage &gt; OnComplete=nullptr) = 0<br> |
| virtual void | [**RequestDestroyVoiceInput**](#function-requestdestroyvoiceinput) (TSharedPtr&lt; [**FTryllVoiceInput**](class_f_tryll_voice_input.md) &gt; Voice) = 0<br> |
| virtual void | [**RequestDownloadModel**](#function-requestdownloadmodel) (const FString & ModelName) = 0<br> |
| virtual void | [**RequestEnsureMutableKb**](#function-requestensuremutablekb) (const FString & Name, const FString & Schema, TFunction&lt; FTryllOnEnsureMutableKb &gt; OnComplete) = 0<br> |
| virtual void | [**RequestEnumerateMutableKbs**](#function-requestenumeratemutablekbs) (TFunction&lt; FTryllOnEnumerateMutableKbs &gt; OnComplete) = 0<br> |
| virtual void | [**RequestGetMemoryConsumption**](#function-requestgetmemoryconsumption) (TFunction&lt; FTryllOnGetMemoryConsumption &gt; OnComplete) = 0<br> |
| virtual void | [**RequestListModels**](#function-requestlistmodels) (TFunction&lt; FTryllOnListModels &gt; OnComplete) = 0<br> |
| virtual void | [**RequestLoadGameMemory**](#function-requestloadgamememory) (const FString & Path, ETryllMutableKbEmbedTiming EmbedTiming, ETryllMutableKbCompatibility Compatibility, TFunction&lt; FTryllOnLoadGameMemory &gt; OnComplete) = 0<br> |
| virtual void | [**RequestLoadModel**](#function-requestloadmodel) (const FString & ModelName, ETryllMemoryPlacement Placement=ETryllMemoryPlacement::Auto, bool bIsOverride=false) = 0<br> |
| virtual void | [**RequestSaveGameMemory**](#function-requestsavegamememory) (const FString & Path, const FString & SaveId, TFunction&lt; FTryllOnSaveGameMemory &gt; OnComplete) = 0<br> |
| virtual void | [**RequestUnloadModel**](#function-requestunloadmodel) (const FString & ModelName) = 0<br> |
| virtual void | [**SetInferenceThrottle**](#function-setinferencethrottle) (float Level) = 0<br> |
| virtual void | [**SetSessionLocale**](#function-setsessionlocale) (FString Locale) = 0<br> |
| virtual bool | [**TryLaunch**](#function-trylaunch) (EServerBuildVariant Variant, int32 KnownPort, int32 IdleShutdownTimeoutSeconds, FString & OutError) = 0<br> |
| virtual void | [**UnregisterPauseListenerProbe**](#function-unregisterpauselistenerprobe) (int32 ProbeId) = 0<br> |
| virtual  | [**~ITryllEditorBridge**](#function-itrylleditorbridge) () = default<br> |




























## Detailed Description


Runtime-facing surface for the editor-only shared Tryll connection/session. UTryllEditorSubsystem (TryllClientEditor) registers an implementation so [**UTryllSubsystem**](class_u_tryll_subsystem.md) (TryllClient) can delegate in PIE without linking the editor module. 


    
## Public Functions Documentation




### function AcquireConnection 

```C++
virtual void ITryllEditorBridge::AcquireConnection () = 0
```




<hr>



### function AdoptRelaunchedServer 

```C++
virtual void ITryllEditorBridge::AdoptRelaunchedServer () = 0
```



After TryLaunch replaced a running server, move an open shared connection onto the new port. No-op when nothing is connected. 


        

<hr>



### function CreateSession 

```C++
virtual void ITryllEditorBridge::CreateSession (
    FString GameName=TEXT(""),
    FString Locale=TEXT("")
) = 0
```



Engines are derived from [**UTryllModelManifest**](class_u_tryll_model_manifest.md), so PIE delegation no longer forwards a per-role engine choice — there is none to forward. Storage root is process-wide (server launch), not a CreateSession argument. 


        

<hr>



### function GetActualPort 

```C++
virtual int32 ITryllEditorBridge::GetActualPort () const = 0
```




<hr>



### function GetOnAgentDestroyedNative 

```C++
virtual FTryllOnAgentDestroyedNative & ITryllEditorBridge::GetOnAgentDestroyedNative () = 0
```




<hr>



### function GetOnConnectionChangedNative 

```C++
virtual FTryllOnConnectionChangedNative & ITryllEditorBridge::GetOnConnectionChangedNative () = 0
```




<hr>



### function GetOnCreateEmbeddedStringStorageCompleteNative 

```C++
virtual FTryllOnCreateEmbeddedStringStorageCompleteNative & ITryllEditorBridge::GetOnCreateEmbeddedStringStorageCompleteNative () = 0
```




<hr>



### function GetOnCreateSessionCompleteNative 

```C++
virtual FTryllOnCreateSessionCompleteNative & ITryllEditorBridge::GetOnCreateSessionCompleteNative () = 0
```




<hr>



### function GetOnCreateStringStorageCompleteNative 

```C++
virtual FTryllOnCreateStringStorageCompleteNative & ITryllEditorBridge::GetOnCreateStringStorageCompleteNative () = 0
```




<hr>



### function GetOnDestroyEmbeddedStringStorageCompleteNative 

```C++
virtual FTryllOnDestroyEmbeddedStringStorageCompleteNative & ITryllEditorBridge::GetOnDestroyEmbeddedStringStorageCompleteNative () = 0
```




<hr>



### function GetOnDestroyStringStorageCompleteNative 

```C++
virtual FTryllOnDestroyStringStorageCompleteNative & ITryllEditorBridge::GetOnDestroyStringStorageCompleteNative () = 0
```




<hr>



### function GetOnDownloadCompleteNative 

```C++
virtual FTryllOnDownloadCompleteNative & ITryllEditorBridge::GetOnDownloadCompleteNative () = 0
```




<hr>



### function GetOnDownloadProgressNative 

```C++
virtual FTryllOnDownloadProgressNative & ITryllEditorBridge::GetOnDownloadProgressNative () = 0
```




<hr>



### function GetOnErrorNative 

```C++
virtual FTryllOnErrorNative & ITryllEditorBridge::GetOnErrorNative () = 0
```




<hr>



### function GetOnGetMemoryConsumptionCompleteNative 

```C++
virtual FTryllOnGetMemoryConsumptionCompleteNative & ITryllEditorBridge::GetOnGetMemoryConsumptionCompleteNative () = 0
```




<hr>



### function GetOnIntentClassifiedNative 

```C++
virtual FTryllOnIntentClassifiedNative & ITryllEditorBridge::GetOnIntentClassifiedNative () = 0
```




<hr>



### function GetOnIntentLlmClassifiedNative 

```C++
virtual FTryllOnIntentLlmClassifiedNative & ITryllEditorBridge::GetOnIntentLlmClassifiedNative () = 0
```




<hr>



### function GetOnListModelsCompleteNative 

```C++
virtual FTryllOnListModelsCompleteNative & ITryllEditorBridge::GetOnListModelsCompleteNative () = 0
```




<hr>



### function GetOnLoadModelCompleteNative 

```C++
virtual FTryllOnLoadModelCompleteNative & ITryllEditorBridge::GetOnLoadModelCompleteNative () = 0
```




<hr>



### function GetOnNodeEventNative 

```C++
virtual FTryllOnNodeEventNative & ITryllEditorBridge::GetOnNodeEventNative () = 0
```




<hr>



### function GetOnPausedForwardNative 

```C++
virtual FTryllOnPausedForwardNative & ITryllEditorBridge::GetOnPausedForwardNative () = 0
```




<hr>



### function GetOnPausedNative 

```C++
virtual FTryllOnPausedNative & ITryllEditorBridge::GetOnPausedNative () = 0
```




<hr>



### function GetOnSetSessionLocaleCompleteNative 

```C++
virtual FTryllOnSetSessionLocaleCompleteNative & ITryllEditorBridge::GetOnSetSessionLocaleCompleteNative () = 0
```




<hr>



### function GetOnToolCallDetailedNative 

```C++
virtual FTryllOnToolCallDetailedNative & ITryllEditorBridge::GetOnToolCallDetailedNative () = 0
```




<hr>



### function GetOnToolCallNative 

```C++
virtual FTryllOnToolCallNative & ITryllEditorBridge::GetOnToolCallNative () = 0
```




<hr>



### function GetOnToolCallWithIdNative 

```C++
virtual FTryllOnToolCallWithIdNative & ITryllEditorBridge::GetOnToolCallWithIdNative () = 0
```




<hr>



### function GetOnUnloadModelCompleteNative 

```C++
virtual FTryllOnUnloadModelCompleteNative & ITryllEditorBridge::GetOnUnloadModelCompleteNative () = 0
```




<hr>



### function GetSessionId 

```C++
virtual int64 ITryllEditorBridge::GetSessionId () const = 0
```




<hr>



### function HasSessionPauseListener 

_Direct OnPausedNative listeners, or any registered probe that returns true._ 
```C++
virtual bool ITryllEditorBridge::HasSessionPauseListener () const = 0
```




<hr>



### function IsConnected 

```C++
virtual bool ITryllEditorBridge::IsConnected () const = 0
```




<hr>



### function IsSessionReady 

```C++
virtual bool ITryllEditorBridge::IsSessionReady () const = 0
```




<hr>



### function RegisterPauseListenerProbe 

```C++
virtual int32 ITryllEditorBridge::RegisterPauseListenerProbe (
    TFunction< bool()> Probe
) = 0
```



Report whether this consumer's session-facing OnPaused is actually bound. Used so a PIE forwarding subscription is not mistaken for a listener. Returns an id to pass to UnregisterPauseListenerProbe. 


        

<hr>



### function ReleaseConnection 

```C++
virtual void ITryllEditorBridge::ReleaseConnection () = 0
```




<hr>



### function RequestCreateAgent 

```C++
virtual void ITryllEditorBridge::RequestCreateAgent (
    const FTryllGraphDescription & Graph,
    TFunction< FTryllOnAgentCreated > OnComplete,
    bool bEnableDiagnostics=false,
    bool bMaintainDialogueHistory=true,
    const TArray< FTryllVariableDecl > & Variables={},
    ETryllKvCacheInitialization KvCacheInitialization=ETryllKvCacheInitialization::AllocateOnly,
    ETryllWorkload Workload=ETryllWorkload::Interactive
) = 0
```




<hr>



### function RequestCreateEmbeddedStringStorage 

```C++
virtual void ITryllEditorBridge::RequestCreateEmbeddedStringStorage (
    const FString & Name,
    const FString & ConfigPath,
    const FString & EmbeddingModel=FString{},
    TFunction< UTryllSubsystem::FTryllOnEmbeddedStorageCreated > OnComplete=nullptr
) = 0
```




<hr>



### function RequestCreateEmbeddedStringStorageFromStrings 

```C++
virtual void ITryllEditorBridge::RequestCreateEmbeddedStringStorageFromStrings (
    const FString & Name,
    const TArray< FString > & Strings,
    const FString & EmbeddingModel,
    TFunction< UTryllSubsystem::FTryllOnEmbeddedStorageCreated > OnComplete=nullptr
) = 0
```




<hr>



### function RequestCreateStringStorage 

```C++
virtual void ITryllEditorBridge::RequestCreateStringStorage (
    const FString & Name,
    const TArray< FString > & Strings,
    TFunction< UTryllSubsystem::FTryllOnStringStorage > OnComplete=nullptr
) = 0
```




<hr>



### function RequestCreateStringStorageFromFile 

```C++
virtual void ITryllEditorBridge::RequestCreateStringStorageFromFile (
    const FString & Name,
    const FString & FilePath,
    uint8 Kind=0,
    TFunction< UTryllSubsystem::FTryllOnStringStorage > OnComplete=nullptr
) = 0
```




<hr>



### function RequestCreateStringStorageKeyed 

```C++
virtual void ITryllEditorBridge::RequestCreateStringStorageKeyed (
    const FString & Name,
    const TArray< FString > & Keys,
    const TArray< FString > & Values,
    uint8 Kind=1,
    TFunction< UTryllSubsystem::FTryllOnStringStorage > OnComplete=nullptr
) = 0
```




<hr>



### function RequestCreateVoiceInput 

```C++
virtual void ITryllEditorBridge::RequestCreateVoiceInput (
    const FTryllVoiceInputConfig & Config,
    TFunction< FTryllOnVoiceInputCreated > OnComplete
) = 0
```




<hr>



### function RequestDestroyAgent 

```C++
virtual void ITryllEditorBridge::RequestDestroyAgent (
    TSharedPtr< FTryllAgent > Agent
) = 0
```




<hr>



### function RequestDestroyEmbeddedStringStorage 

```C++
virtual void ITryllEditorBridge::RequestDestroyEmbeddedStringStorage (
    const FString & Name,
    TFunction< UTryllSubsystem::FTryllOnEmbeddedStorageDestroyed > OnComplete=nullptr
) = 0
```




<hr>



### function RequestDestroyStringStorage 

```C++
virtual void ITryllEditorBridge::RequestDestroyStringStorage (
    const FString & Name,
    TFunction< UTryllSubsystem::FTryllOnStringStorage > OnComplete=nullptr
) = 0
```




<hr>



### function RequestDestroyVoiceInput 

```C++
virtual void ITryllEditorBridge::RequestDestroyVoiceInput (
    TSharedPtr< FTryllVoiceInput > Voice
) = 0
```




<hr>



### function RequestDownloadModel 

```C++
virtual void ITryllEditorBridge::RequestDownloadModel (
    const FString & ModelName
) = 0
```




<hr>



### function RequestEnsureMutableKb 

```C++
virtual void ITryllEditorBridge::RequestEnsureMutableKb (
    const FString & Name,
    const FString & Schema,
    TFunction< FTryllOnEnsureMutableKb > OnComplete
) = 0
```




<hr>



### function RequestEnumerateMutableKbs 

```C++
virtual void ITryllEditorBridge::RequestEnumerateMutableKbs (
    TFunction< FTryllOnEnumerateMutableKbs > OnComplete
) = 0
```




<hr>



### function RequestGetMemoryConsumption 

```C++
virtual void ITryllEditorBridge::RequestGetMemoryConsumption (
    TFunction< FTryllOnGetMemoryConsumption > OnComplete
) = 0
```




<hr>



### function RequestListModels 

```C++
virtual void ITryllEditorBridge::RequestListModels (
    TFunction< FTryllOnListModels > OnComplete
) = 0
```




<hr>



### function RequestLoadGameMemory 

```C++
virtual void ITryllEditorBridge::RequestLoadGameMemory (
    const FString & Path,
    ETryllMutableKbEmbedTiming EmbedTiming,
    ETryllMutableKbCompatibility Compatibility,
    TFunction< FTryllOnLoadGameMemory > OnComplete
) = 0
```




<hr>



### function RequestLoadModel 

```C++
virtual void ITryllEditorBridge::RequestLoadModel (
    const FString & ModelName,
    ETryllMemoryPlacement Placement=ETryllMemoryPlacement::Auto,
    bool bIsOverride=false
) = 0
```



`Placement` is honoured only when `bIsOverride` is true; false means "use
the session's per-model hint". 


        

<hr>



### function RequestSaveGameMemory 

```C++
virtual void ITryllEditorBridge::RequestSaveGameMemory (
    const FString & Path,
    const FString & SaveId,
    TFunction< FTryllOnSaveGameMemory > OnComplete
) = 0
```




<hr>



### function RequestUnloadModel 

```C++
virtual void ITryllEditorBridge::RequestUnloadModel (
    const FString & ModelName
) = 0
```




<hr>



### function SetInferenceThrottle 

```C++
virtual void ITryllEditorBridge::SetInferenceThrottle (
    float Level
) = 0
```



Fire-and-forget — see [**UTryllSubsystem::SetInferenceThrottle**](class_u_tryll_subsystem.md#function-setinferencethrottle). 


        

<hr>



### function SetSessionLocale 

```C++
virtual void ITryllEditorBridge::SetSessionLocale (
    FString Locale
) = 0
```




<hr>



### function TryLaunch 

```C++
virtual bool ITryllEditorBridge::TryLaunch (
    EServerBuildVariant Variant,
    int32 KnownPort,
    int32 IdleShutdownTimeoutSeconds,
    FString & OutError
) = 0
```




<hr>



### function UnregisterPauseListenerProbe 

```C++
virtual void ITryllEditorBridge::UnregisterPauseListenerProbe (
    int32 ProbeId
) = 0
```




<hr>



### function ~ITryllEditorBridge 

```C++
virtual ITryllEditorBridge::~ITryllEditorBridge () = default
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllEditorBridge.h`

