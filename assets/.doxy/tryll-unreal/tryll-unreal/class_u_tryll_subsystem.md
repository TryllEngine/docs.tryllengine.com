

# Class UTryllSubsystem



[**ClassList**](annotated.md) **>** [**UTryllSubsystem**](class_u_tryll_subsystem.md)



[More...](#detailed-description)

* `#include <TryllSubsystem.h>`



Inherits the following classes: UGameInstanceSubsystem,  FTickableGameObject












## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FEmbeddedStorageInfo**](struct_u_tryll_subsystem_1_1_f_embedded_storage_info.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| typedef void([**FEmbeddedStorageInfo**](struct_u_tryll_subsystem_1_1_f_embedded_storage_info.md), [**FTryllError**](struct_f_tryll_error.md)) | [**FTryllOnEmbeddedStorageCreated**](#typedef-ftryllonembeddedstoragecreated)  <br> |
| typedef void([**FTryllError**](struct_f_tryll_error.md)) | [**FTryllOnEmbeddedStorageDestroyed**](#typedef-ftryllonembeddedstoragedestroyed)  <br> |
| typedef void([**FTryllError**](struct_f_tryll_error.md)) | [**FTryllOnStringStorage**](#typedef-ftryllonstringstorage)  <br> |




## Public Attributes

| Type | Name |
| ---: | :--- |
|  FOnTryllAgentDestroyed | [**OnAgentDestroyed**](#variable-onagentdestroyed)  <br> |
|  FOnTryllConnectionChanged | [**OnConnectionChanged**](#variable-onconnectionchanged)  <br> |
|  FOnTryllCreateEmbeddedStringStorage | [**OnCreateEmbeddedStringStorageComplete**](#variable-oncreateembeddedstringstoragecomplete)  <br> |
|  FOnTryllCreateSession | [**OnCreateSessionComplete**](#variable-oncreatesessioncomplete)  <br> |
|  FOnTryllCreateStringStorage | [**OnCreateStringStorageComplete**](#variable-oncreatestringstoragecomplete)  <br> |
|  FOnTryllDestroyEmbeddedStringStorage | [**OnDestroyEmbeddedStringStorageComplete**](#variable-ondestroyembeddedstringstoragecomplete)  <br> |
|  FOnTryllDestroyStringStorage | [**OnDestroyStringStorageComplete**](#variable-ondestroystringstoragecomplete)  <br> |
|  FOnTryllDownloadComplete | [**OnDownloadComplete**](#variable-ondownloadcomplete)  <br> |
|  FOnTryllDownloadProgress | [**OnDownloadProgress**](#variable-ondownloadprogress)  <br> |
|  FOnTryllError | [**OnError**](#variable-onerror)  <br> |
|  FOnTryllIntentClassified | [**OnIntentClassified**](#variable-onintentclassified)  <br> |
|  FOnTryllListModels | [**OnListModelsComplete**](#variable-onlistmodelscomplete)  <br> |
|  FOnTryllLoadModel | [**OnLoadModelComplete**](#variable-onloadmodelcomplete)  <br> |
|  FOnTryllModelReady | [**OnModelReady**](#variable-onmodelready)  <br> |
|  FOnTryllNodeEvent | [**OnNodeEvent**](#variable-onnodeevent)  <br> |
|  FOnTryllPaused | [**OnPaused**](#variable-onpaused)  <br> |
|  FOnTryllToolCall | [**OnToolCall**](#variable-ontoolcall)  <br> |
|  FOnTryllToolCallDetailed | [**OnToolCallDetailed**](#variable-ontoolcalldetailed)  <br> |
|  FOnTryllToolCallWithId | [**OnToolCallWithId**](#variable-ontoolcallwithid)  <br> |
|  FOnTryllUnloadModel | [**OnUnloadModelComplete**](#variable-onunloadmodelcomplete)  <br> |
|  FString | [**ServerBuildConfig**](#variable-serverbuildconfig)  <br> |
|  FString | [**ServerExePath**](#variable-serverexepath)  <br> |
|  FString | [**ServerHost**](#variable-serverhost)   = `TEXT("127.0.0.1")`<br> |
|  int32 | [**ServerIdleTimeoutSeconds**](#variable-serveridletimeoutseconds)   = `0`<br> |
|  int32 | [**ServerPort**](#variable-serverport)   = `9100`<br> |
|  FString | [**ServerRunId**](#variable-serverrunid)  <br> |
|  FString | [**ServerVersion**](#variable-serverversion)  <br> |
|  bool | [**bServerManaged**](#variable-bservermanaged)   = `false`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  void | [**Connect**](#function-connect) () <br> |
|  void | [**CreateSession**](#function-createsession) (ETryllInferenceEngine Engine, FString GameName=TEXT(""), ETryllInferenceEngine SttEngine=ETryllInferenceEngine::Mock, ETryllInferenceEngine TtsEngine=ETryllInferenceEngine::Mock, ETryllInferenceEngine EmbeddingEngine=ETryllInferenceEngine::Mock, FString StorageDataFolder=TEXT("")) <br> |
|  void | [**CreateSessionFromSettings**](#function-createsessionfromsettings) () <br> |
| virtual void | [**Deinitialize**](#function-deinitialize) () override<br> |
|  void | [**Disconnect**](#function-disconnect) () <br> |
|  int64 | [**GetSessionId**](#function-getsessionid) () const<br> |
| virtual TStatId | [**GetStatId**](#function-getstatid) () override const<br> |
| virtual ETickableTickType | [**GetTickableTickType**](#function-gettickableticktype) () override const<br> |
| virtual void | [**Initialize**](#function-initialize) (FSubsystemCollectionBase & Collection) override<br> |
|  bool | [**IsConnected**](#function-isconnected) () const<br> |
|  bool | [**IsSessionReady**](#function-issessionready) () const<br> |
| virtual bool | [**IsTickable**](#function-istickable) () override const<br> |
| virtual bool | [**IsTickableInEditor**](#function-istickableineditor) () override const<br> |
| virtual bool | [**IsTickableWhenPaused**](#function-istickablewhenpaused) () override const<br> |
|  void | [**RegisterAgent**](#function-registeragent) (std::uint64\_t AgentId, TWeakPtr&lt; [**FTryllAgent**](class_f_tryll_agent.md) &gt; Agent) <br> |
|  void | [**RequestCreateAgent**](#function-requestcreateagent) (const [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) & Graph, TFunction&lt; FTryllOnAgentCreated &gt; OnComplete, bool bEnableDiagnostics=false, bool bMaintainDialogueHistory=true, const TArray&lt; [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) &gt; & Variables={}, ETryllKvCacheInitialization KvCacheInitialization=ETryllKvCacheInitialization::AllocateOnly) <br> |
|  void | [**RequestCreateEmbeddedStringStorage**](#function-requestcreateembeddedstringstorage) (const FString & Name, const FString & ConfigPath, const FString & EmbeddingModel=FString{}, TFunction&lt; FTryllOnEmbeddedStorageCreated &gt; OnComplete=nullptr) <br> |
|  void | [**RequestCreateEmbeddedStringStorageFromStrings**](#function-requestcreateembeddedstringstoragefromstrings) (const FString & Name, const TArray&lt; FString &gt; & Strings, const FString & EmbeddingModel, TFunction&lt; FTryllOnEmbeddedStorageCreated &gt; OnComplete=nullptr) <br> |
|  void | [**RequestCreateStringStorage**](#function-requestcreatestringstorage) (const FString & Name, const TArray&lt; FString &gt; & Strings, TFunction&lt; FTryllOnStringStorage &gt; OnComplete=nullptr) <br> |
|  void | [**RequestCreateStringStorageFromFile**](#function-requestcreatestringstoragefromfile) (const FString & Name, const FString & FilePath, uint8 Kind=0, TFunction&lt; FTryllOnStringStorage &gt; OnComplete=nullptr) <br> |
|  void | [**RequestCreateStringStorageKeyed**](#function-requestcreatestringstoragekeyed) (const FString & Name, const TArray&lt; FString &gt; & Keys, const TArray&lt; FString &gt; & Values, uint8 Kind=1, TFunction&lt; FTryllOnStringStorage &gt; OnComplete=nullptr) <br> |
|  void | [**RequestCreateVoiceInput**](#function-requestcreatevoiceinput) (const [**FTryllVoiceInputConfig**](struct_f_tryll_voice_input_config.md) & Config, TFunction&lt; FTryllOnVoiceInputCreated &gt; OnComplete) <br> |
|  void | [**RequestDestroyAgent**](#function-requestdestroyagent) (TSharedPtr&lt; [**FTryllAgent**](class_f_tryll_agent.md) &gt; Agent) <br> |
|  void | [**RequestDestroyEmbeddedStringStorage**](#function-requestdestroyembeddedstringstorage) (const FString & Name, TFunction&lt; FTryllOnEmbeddedStorageDestroyed &gt; OnComplete=nullptr) <br> |
|  void | [**RequestDestroyStringStorage**](#function-requestdestroystringstorage) (const FString & Name, TFunction&lt; FTryllOnStringStorage &gt; OnComplete=nullptr) <br> |
|  void | [**RequestDestroyVoiceInput**](#function-requestdestroyvoiceinput) (TSharedPtr&lt; [**FTryllVoiceInput**](class_f_tryll_voice_input.md) &gt; Voice) <br> |
|  void | [**RequestDownloadModel**](#function-requestdownloadmodel) (const FString & ModelName) <br> |
|  void | [**RequestListModels**](#function-requestlistmodels) (TFunction&lt; FTryllOnListModels &gt; OnComplete) <br> |
|  void | [**RequestLoadModel**](#function-requestloadmodel) (const FString & ModelName) <br> |
|  void | [**RequestModel**](#function-requestmodel) (const FString & ModelName) <br> |
|  void | [**RequestUnloadModel**](#function-requestunloadmodel) (const FString & ModelName) <br> |
| virtual void | [**Tick**](#function-tick) (float DeltaTime) override<br> |
|  void | [**UnregisterAgent**](#function-unregisteragent) (std::uint64\_t AgentId) <br> |


## Public Static Functions

| Type | Name |
| ---: | :--- |
|  TArray&lt; EServerBuildVariant &gt; | [**GetAvailableServerVariants**](#function-getavailableservervariants) () <br> |
|  FString | [**GetServerRootDir**](#function-getserverrootdir) () <br> |
|  FString | [**ResolveServerExePath**](#function-resolveserverexepath) (EServerBuildVariant Preferred) <br> |


























## Detailed Description


Game-instance subsystem: one TCP session to the Tryll server. Manages the socket connection on a background thread (FTryllConnection) and drains the event queue on Tick(), dispatching to registered agents and delegates.


Default (zero-wiring) path — driven by [**UTryllRuntimeSettings**](class_u_tryll_runtime_settings.md):
* bAutoLaunchServer → spawns the bundled tryll\_server.exe on Initialize.
* bAutoConnect → [**Connect()**](class_u_tryll_subsystem.md#function-connect) on Initialize (connect-retry covers server startup).
* bAutoCreateSession → on Connected, [**CreateSessionFromSettings()**](class_u_tryll_subsystem.md#function-createsessionfromsettings) applies the Engine / Stt / Tts / Embedding selections from settings. [**UTryllAgentComponent**](class_u_tryll_agent_component.md) / [**UTryllVoiceInputComponent**](class_u_tryll_voice_input_component.md) then create themselves once [**IsSessionReady()**](class_u_tryll_subsystem.md#function-issessionready) — they queue until OnCreateSessionComplete if they spawn earlier. So the common case is: add the components, pick engines in Project Settings, press play.




Manual path — turn those flags off to drive the session yourself:
* [**Connect()**](class_u_tryll_subsystem.md#function-connect) — open TCP session to ServerHost:ServerPort.
* OnConnectionChanged(bConnected=true) — bind to continue; or poll [**IsConnected()**](class_u_tryll_subsystem.md#function-isconnected).
* CreateSession(Engine, ...) — configure explicitly (overrides settings). Completion arrives on OnCreateSessionComplete; [**IsSessionReady()**](class_u_tryll_subsystem.md#function-issessionready) is true thereafter.
* CreateAgent — attach a [**UTryllAgentComponent**](class_u_tryll_agent_component.md) (auto-creates once the session is ready) or call [**RequestCreateAgent()**](class_u_tryll_subsystem.md#function-requestcreateagent).
* SendMessage(Text) — drive one turn; chunks on OnAnswerText, the turn ends with OnTurnComplete(Status).
* RequestDestroyAgent / Disconnect — tear down when done.




All delegates fire on the game thread from Tick(); it is safe to touch UObjects and UI directly from callbacks. 


    
## Public Types Documentation




### typedef FTryllOnEmbeddedStorageCreated 

```C++
using UTryllSubsystem::FTryllOnEmbeddedStorageCreated = void(FEmbeddedStorageInfo, FTryllError);
```




<hr>



### typedef FTryllOnEmbeddedStorageDestroyed 

```C++
using UTryllSubsystem::FTryllOnEmbeddedStorageDestroyed = void(FTryllError);
```




<hr>



### typedef FTryllOnStringStorage 

```C++
using UTryllSubsystem::FTryllOnStringStorage = void(FTryllError);
```




<hr>
## Public Attributes Documentation




### variable OnAgentDestroyed 

```C++
FOnTryllAgentDestroyed UTryllSubsystem::OnAgentDestroyed;
```




<hr>



### variable OnConnectionChanged 

```C++
FOnTryllConnectionChanged UTryllSubsystem::OnConnectionChanged;
```




<hr>



### variable OnCreateEmbeddedStringStorageComplete 

```C++
FOnTryllCreateEmbeddedStringStorage UTryllSubsystem::OnCreateEmbeddedStringStorageComplete;
```




<hr>



### variable OnCreateSessionComplete 

```C++
FOnTryllCreateSession UTryllSubsystem::OnCreateSessionComplete;
```




<hr>



### variable OnCreateStringStorageComplete 

```C++
FOnTryllCreateStringStorage UTryllSubsystem::OnCreateStringStorageComplete;
```




<hr>



### variable OnDestroyEmbeddedStringStorageComplete 

```C++
FOnTryllDestroyEmbeddedStringStorage UTryllSubsystem::OnDestroyEmbeddedStringStorageComplete;
```




<hr>



### variable OnDestroyStringStorageComplete 

```C++
FOnTryllDestroyStringStorage UTryllSubsystem::OnDestroyStringStorageComplete;
```




<hr>



### variable OnDownloadComplete 

```C++
FOnTryllDownloadComplete UTryllSubsystem::OnDownloadComplete;
```




<hr>



### variable OnDownloadProgress 

```C++
FOnTryllDownloadProgress UTryllSubsystem::OnDownloadProgress;
```




<hr>



### variable OnError 

```C++
FOnTryllError UTryllSubsystem::OnError;
```




<hr>



### variable OnIntentClassified 

```C++
FOnTryllIntentClassified UTryllSubsystem::OnIntentClassified;
```



Broadcast when a ClassifyIntentNode emits an intent on its "found" path with notify\_client enabled. 


        

<hr>



### variable OnListModelsComplete 

```C++
FOnTryllListModels UTryllSubsystem::OnListModelsComplete;
```




<hr>



### variable OnLoadModelComplete 

```C++
FOnTryllLoadModel UTryllSubsystem::OnLoadModelComplete;
```




<hr>



### variable OnModelReady 

```C++
FOnTryllModelReady UTryllSubsystem::OnModelReady;
```



Fired when a RequestModel sequence terminates (load succeeded, or fallback download+load succeeded, or any step failed). bSuccess is the final outcome. 


        

<hr>



### variable OnNodeEvent 

```C++
FOnTryllNodeEvent UTryllSubsystem::OnNodeEvent;
```



Broadcast for any NodeEvent whose event\_type has no typed delegate (or whose typed delegate has no subscribers). Untyped fallback. 


        

<hr>



### variable OnPaused 

```C++
FOnTryllPaused UTryllSubsystem::OnPaused;
```



Broadcast when the executor pauses a turn between nodes (Pause node, or pausing ToolCall dispositions). 


        

<hr>



### variable OnToolCall 

```C++
FOnTryllToolCall UTryllSubsystem::OnToolCall;
```



Broadcast when a ToolCallNode emits a tool\_call NodeEvent (notifying dispositions). 


        

<hr>



### variable OnToolCallDetailed 

```C++
FOnTryllToolCallDetailed UTryllSubsystem::OnToolCallDetailed;
```



Detailed tool-call event carrying the full owning [**FTryllToolCall**](struct_f_tryll_tool_call.md) (node + disposition). 


        

<hr>



### variable OnToolCallWithId 

```C++
FOnTryllToolCallWithId UTryllSubsystem::OnToolCallWithId;
```



Call-ID-aware tool-call event used to correlate Resume tool results. 


        

<hr>



### variable OnUnloadModelComplete 

```C++
FOnTryllUnloadModel UTryllSubsystem::OnUnloadModelComplete;
```




<hr>



### variable ServerBuildConfig 

```C++
FString UTryllSubsystem::ServerBuildConfig;
```



Server build configuration ("Debug" \| "Release" \| "Production"). 


        

<hr>



### variable ServerExePath 

```C++
FString UTryllSubsystem::ServerExePath;
```



Canonical path of the connected server executable (empty if unknown). 


        

<hr>



### variable ServerHost 

```C++
FString UTryllSubsystem::ServerHost;
```



Host name or IP address of the Tryll server. Defaults to localhost. 


        

<hr>



### variable ServerIdleTimeoutSeconds 

```C++
int32 UTryllSubsystem::ServerIdleTimeoutSeconds;
```



Server idle-shutdown grace period in seconds (0 when standalone). 


        

<hr>



### variable ServerPort 

```C++
int32 UTryllSubsystem::ServerPort;
```



TCP port the Tryll server is listening on (default: 9100). 


        

<hr>



### variable ServerRunId 

```C++
FString UTryllSubsystem::ServerRunId;
```



Per-process server run id (UUID). 


        

<hr>



### variable ServerVersion 

```C++
FString UTryllSubsystem::ServerVersion;
```



Server version string. 


        

<hr>



### variable bServerManaged 

```C++
bool UTryllSubsystem::bServerManaged;
```



True when the server runs in managed (idle-self-exit) mode. 


        

<hr>
## Public Functions Documentation




### function Connect 

```C++
void UTryllSubsystem::Connect () 
```



Open a TCP session to ServerHost:ServerPort. Asynchronous — completion is reported via OnConnectionChanged(bConnected=true). Errors fire OnError. Calling [**Connect()**](class_u_tryll_subsystem.md#function-connect) while already connected is a no-op. 


        

<hr>



### function CreateSession 

```C++
void UTryllSubsystem::CreateSession (
    ETryllInferenceEngine Engine,
    FString GameName=TEXT(""),
    ETryllInferenceEngine SttEngine=ETryllInferenceEngine::Mock,
    ETryllInferenceEngine TtsEngine=ETryllInferenceEngine::Mock,
    ETryllInferenceEngine EmbeddingEngine=ETryllInferenceEngine::Mock,
    FString StorageDataFolder=TEXT("")
) 
```



Send CreateSessionRequest. Must be called after Connected and before CreateAgent. Result is delivered via OnCreateSessionComplete delegate.




**Parameters:**


* `Engine` Inference backend to use for this session. 
* `GameName` Integration identifier for telemetry grouping (e.g. "tryll-roleplay-demo"). Leave empty to omit; the server defaults to "unknown" when telemetry is on. Pass [**UTryllRuntimeSettings::GameName**](class_u_tryll_runtime_settings.md#variable-gamename) to drive this from the project settings asset. 
* `StorageDataFolder` Project-relative storage root (e.g. [**UTryllRuntimeSettings::StorageDataFolder**](class_u_tryll_runtime_settings.md#variable-storagedatafolder)) or an absolute path. Resolved to a full path on the wire. Empty = server default storage root. 




        

<hr>



### function CreateSessionFromSettings 

```C++
void UTryllSubsystem::CreateSessionFromSettings () 
```



Configure the session from project settings ([**UTryllRuntimeSettings**](class_u_tryll_runtime_settings.md)): engine selections, game name, and storage folder.


This is the "use settings" path. It runs automatically once connected when [**UTryllRuntimeSettings::bAutoCreateSession**](class_u_tryll_runtime_settings.md#variable-bautocreatesession) is true; call it yourself if you connected with that flag disabled and still want the settings-driven config. To override settings, call [**CreateSession()**](class_u_tryll_subsystem.md#function-createsession) with explicit parameters instead. 


        

<hr>



### function Deinitialize 

```C++
virtual void UTryllSubsystem::Deinitialize () override
```




<hr>



### function Disconnect 

```C++
void UTryllSubsystem::Disconnect () 
```



Close the active session. OnConnectionChanged(bConnected=false) fires once the socket is down. 


        

<hr>



### function GetSessionId 

```C++
int64 UTryllSubsystem::GetSessionId () const
```



Server-assigned session id, or 0 when not connected. 


        

<hr>



### function GetStatId 

```C++
virtual TStatId UTryllSubsystem::GetStatId () override const
```




<hr>



### function GetTickableTickType 

```C++
virtual ETickableTickType UTryllSubsystem::GetTickableTickType () override const
```




<hr>



### function Initialize 

```C++
virtual void UTryllSubsystem::Initialize (
    FSubsystemCollectionBase & Collection
) override
```




<hr>



### function IsConnected 

```C++
bool UTryllSubsystem::IsConnected () const
```



True while the background connection is in the Connected state. 


        

<hr>



### function IsSessionReady 

```C++
bool UTryllSubsystem::IsSessionReady () const
```



True once the current connection has a successfully configured session (CreateSession completed OK). Agent / VoiceInput creation must wait for this — creating against an unconfigured session is rejected by the server. 


        

<hr>



### function IsTickable 

```C++
virtual bool UTryllSubsystem::IsTickable () override const
```




<hr>



### function IsTickableInEditor 

```C++
inline virtual bool UTryllSubsystem::IsTickableInEditor () override const
```




<hr>



### function IsTickableWhenPaused 

```C++
inline virtual bool UTryllSubsystem::IsTickableWhenPaused () override const
```




<hr>



### function RegisterAgent 

```C++
void UTryllSubsystem::RegisterAgent (
    std::uint64_t AgentId,
    TWeakPtr< FTryllAgent > Agent
) 
```




<hr>



### function RequestCreateAgent 

```C++
void UTryllSubsystem::RequestCreateAgent (
    const FTryllGraphDescription & Graph,
    TFunction< FTryllOnAgentCreated > OnComplete,
    bool bEnableDiagnostics=false,
    bool bMaintainDialogueHistory=true,
    const TArray< FTryllVariableDecl > & Variables={},
    ETryllKvCacheInitialization KvCacheInitialization=ETryllKvCacheInitialization::AllocateOnly
) 
```



Begin create-agent; completion is delivered via OnComplete on the game thread. 

**Parameters:**


* `Graph` Workflow graph description to send to the server. 
* `bEnableDiagnostics` When true the server populates TurnComplete.debug\_info. 
* `bMaintainDialogueHistory` When true (default) the agent keeps its full dialogue history; when false the history is discarded after each turn (stateless agent — for classification/routing/tool-picking agents). 




        

<hr>



### function RequestCreateEmbeddedStringStorage 

```C++
void UTryllSubsystem::RequestCreateEmbeddedStringStorage (
    const FString & Name,
    const FString & ConfigPath,
    const FString & EmbeddingModel=FString{},
    TFunction< FTryllOnEmbeddedStorageCreated > OnComplete=nullptr
) 
```



Create an EmbeddedStringStorage from a server-side \*.fig.json. Path A: config file + optional embeddingModel override. 


        

<hr>



### function RequestCreateEmbeddedStringStorageFromStrings 

```C++
void UTryllSubsystem::RequestCreateEmbeddedStringStorageFromStrings (
    const FString & Name,
    const TArray< FString > & Strings,
    const FString & EmbeddingModel,
    TFunction< FTryllOnEmbeddedStorageCreated > OnComplete=nullptr
) 
```



Create an EmbeddedStringStorage from inline strings. Path B: requires EmbeddingModel. 


        

<hr>



### function RequestCreateStringStorage 

```C++
void UTryllSubsystem::RequestCreateStringStorage (
    const FString & Name,
    const TArray< FString > & Strings,
    TFunction< FTryllOnStringStorage > OnComplete=nullptr
) 
```



Create a named StringStorage on the server from an inline string array. Completion is broadcast via OnCreateStringStorageComplete. 


        

<hr>



### function RequestCreateStringStorageFromFile 

```C++
void UTryllSubsystem::RequestCreateStringStorageFromFile (
    const FString & Name,
    const FString & FilePath,
    uint8 Kind=0,
    TFunction< FTryllOnStringStorage > OnComplete=nullptr
) 
```



Create a named StringStorage on the server from a server-side file path. 

**Parameters:**


* `Kind` 0=List (default), 1=Map, 2=Multimap. When Kind is Map or Multimap the file must be a JSON array of {id, text} objects. Completion is broadcast via OnCreateStringStorageComplete. 




        

<hr>



### function RequestCreateStringStorageKeyed 

```C++
void UTryllSubsystem::RequestCreateStringStorageKeyed (
    const FString & Name,
    const TArray< FString > & Keys,
    const TArray< FString > & Values,
    uint8 Kind=1,
    TFunction< FTryllOnStringStorage > OnComplete=nullptr
) 
```



Create a named StringStorage on the server from inline key/value pairs. 

**Parameters:**


* `Kind` 1=Map (default; keys must be unique), 2=Multimap (duplicate keys allowed). Completion is broadcast via OnCreateStringStorageComplete. 




        

<hr>



### function RequestCreateVoiceInput 

```C++
void UTryllSubsystem::RequestCreateVoiceInput (
    const FTryllVoiceInputConfig & Config,
    TFunction< FTryllOnVoiceInputCreated > OnComplete
) 
```



Create a server-side VoiceInput (STT) handle. Completion is delivered via OnComplete on the game thread with a TSharedPtr&lt;FTryllVoiceInput&gt; on success. 


        

<hr>



### function RequestDestroyAgent 

```C++
void UTryllSubsystem::RequestDestroyAgent (
    TSharedPtr< FTryllAgent > Agent
) 
```



Queue destroy for the given agent. 


        

<hr>



### function RequestDestroyEmbeddedStringStorage 

```C++
void UTryllSubsystem::RequestDestroyEmbeddedStringStorage (
    const FString & Name,
    TFunction< FTryllOnEmbeddedStorageDestroyed > OnComplete=nullptr
) 
```



Destroy a named EmbeddedStringStorage on the server. Nodes that already hold the storage are unaffected. 


        

<hr>



### function RequestDestroyStringStorage 

```C++
void UTryllSubsystem::RequestDestroyStringStorage (
    const FString & Name,
    TFunction< FTryllOnStringStorage > OnComplete=nullptr
) 
```



Destroy a named StringStorage on the server. Nodes that already hold the storage are unaffected. 


        

<hr>



### function RequestDestroyVoiceInput 

```C++
void UTryllSubsystem::RequestDestroyVoiceInput (
    TSharedPtr< FTryllVoiceInput > Voice
) 
```



Queue destroy for the given VoiceInput handle. 


        

<hr>



### function RequestDownloadModel 

```C++
void UTryllSubsystem::RequestDownloadModel (
    const FString & ModelName
) 
```



Start downloading a model. Progress is broadcast via OnDownloadProgress (name, bytesDownloaded, totalBytes, percent 0–100); completion via OnDownloadComplete (name, bSuccess, errorMessage — empty on success). 


        

<hr>



### function RequestListModels 

```C++
void UTryllSubsystem::RequestListModels (
    TFunction< FTryllOnListModels > OnComplete
) 
```



List models available for the session's inference engine. 


        

<hr>



### function RequestLoadModel 

```C++
void UTryllSubsystem::RequestLoadModel (
    const FString & ModelName
) 
```



Pin a model into memory on the server (LoadAndPin). Completion is broadcast via OnLoadModelComplete. 


        

<hr>



### function RequestModel 

```C++
void UTryllSubsystem::RequestModel (
    const FString & ModelName
) 
```



Ensure a model is loaded and pinned. Tries LoadModel first; if the model is not present on disk the subsystem falls back to DownloadModel and retries the load. Single completion event: [**OnModelReady(name, bSuccess)**](class_u_tryll_subsystem.md#variable-onmodelready). 


        

<hr>



### function RequestUnloadModel 

```C++
void UTryllSubsystem::RequestUnloadModel (
    const FString & ModelName
) 
```



Demote a pinned model to on-demand and evict it if no contexts hold it. Completion is broadcast via OnUnloadModelComplete. 


        

<hr>



### function Tick 

```C++
virtual void UTryllSubsystem::Tick (
    float DeltaTime
) override
```




<hr>



### function UnregisterAgent 

```C++
void UTryllSubsystem::UnregisterAgent (
    std::uint64_t AgentId
) 
```




<hr>
## Public Static Functions Documentation




### function GetAvailableServerVariants 

```C++
static TArray< EServerBuildVariant > UTryllSubsystem::GetAvailableServerVariants () 
```



Variant subfolders (Default/Release) whose target resolves on disk. 


        

<hr>



### function GetServerRootDir 

```C++
static FString UTryllSubsystem::GetServerRootDir () 
```



&lt;PluginBaseDir&gt;/Binaries/ThirdParty/TryllServer; empty if the plugin is not found. 


        

<hr>



### function ResolveServerExePath 

```C++
static FString UTryllSubsystem::ResolveServerExePath (
    EServerBuildVariant Preferred
) 
```



Server exe to launch for Preferred; see .cpp for the editor/packaged resolution. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllSubsystem.h`

