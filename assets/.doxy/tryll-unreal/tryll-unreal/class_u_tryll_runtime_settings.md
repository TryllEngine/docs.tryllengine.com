

# Class UTryllRuntimeSettings



[**ClassList**](annotated.md) **>** [**UTryllRuntimeSettings**](class_u_tryll_runtime_settings.md)



[More...](#detailed-description)

* `#include <TryllRuntimeSettings.h>`



Inherits the following classes: UDeveloperSettings


















## Public Attributes

| Type | Name |
| ---: | :--- |
|  int32 | [**ConnectMaxAttempts**](#variable-connectmaxattempts)   = `5`<br> |
|  float | [**ConnectRetryDelaySeconds**](#variable-connectretrydelayseconds)   = `1.0f`<br> |
|  ETryllInferenceEngine | [**EmbeddingEngine**](#variable-embeddingengine)   = `ETryllInferenceEngine::Mock`<br> |
|  ETryllInferenceEngine | [**Engine**](#variable-engine)   = `ETryllInferenceEngine::LlamaCpp`<br> |
|  FString | [**GameName**](#variable-gamename)  <br> |
|  EServerBuildVariant | [**ServerBuildVariant**](#variable-serverbuildvariant)   = `EServerBuildVariant::Default`<br> |
|  FString | [**StorageDataFolder**](#variable-storagedatafolder)  <br> |
|  ETryllInferenceEngine | [**SttEngine**](#variable-sttengine)   = `ETryllInferenceEngine::Mock`<br> |
|  ETryllInferenceEngine | [**TtsEngine**](#variable-ttsengine)   = `ETryllInferenceEngine::Mock`<br> |
|  bool | [**bAllowAutoModelDownloading**](#variable-ballowautomodeldownloading)   = `false`<br> |
|  bool | [**bAutoConfigureSession**](#variable-bautoconfiguresession)   = `true`<br> |
|  bool | [**bAutoConnect**](#variable-bautoconnect)   = `true`<br> |
|  bool | [**bAutoLaunchServer**](#variable-bautolaunchserver)   = `true`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
| virtual FName | [**GetCategoryName**](#function-getcategoryname) () override const<br> |
|  FString | [**ResolveStorageDataFolderForServer**](#function-resolvestoragedatafolderforserver-12) () const<br> |


## Public Static Functions

| Type | Name |
| ---: | :--- |
|  FString | [**GetConfiguredStorageDataFolder**](#function-getconfiguredstoragedatafolder) () <br> |
|  FString | [**GetStorageRootAbsolute**](#function-getstoragerootabsolute) () <br> |
|  bool | [**MakePathRelativeToStorageRoot**](#function-makepathrelativetostorageroot) (const FString & AbsoluteFilePath, FString & OutRelativePath) <br> |
|  FString | [**NormalizeStorageRelativePath**](#function-normalizestoragerelativepath) (const FString & RelativePath) <br> |
|  FString | [**ResolveStorageDataFolderForServer**](#function-resolvestoragedatafolderforserver-22) (const FString & ConfiguredFolder) <br> |


























## Detailed Description


Project-level settings for the Tryll Client plugin. Edit in Project Settings → Plugins → Tryll Client. Persisted to &lt;YourProject&gt;/Config/DefaultGame.ini under [/Script/TryllClient.TryllRuntimeSettings]. 


    
## Public Attributes Documentation




### variable ConnectMaxAttempts 

```C++
int32 UTryllRuntimeSettings::ConnectMaxAttempts;
```



Max number of attempts when establishing a session via [**UTryllSubsystem::Connect**](class_u_tryll_subsystem.md#function-connect). 1 disables retry. The first call counts as attempt 1. 


        

<hr>



### variable ConnectRetryDelaySeconds 

```C++
float UTryllRuntimeSettings::ConnectRetryDelaySeconds;
```



Constant delay between connect attempts. 


        

<hr>



### variable EmbeddingEngine 

```C++
ETryllInferenceEngine UTryllRuntimeSettings::EmbeddingEngine;
```



Embedding backend for string-storage / RAG nodes. 


        

<hr>



### variable Engine 

```C++
ETryllInferenceEngine UTryllRuntimeSettings::Engine;
```



LLM inference backend for generate / tool-call nodes. 


        

<hr>



### variable GameName 

```C++
FString UTryllRuntimeSettings::GameName;
```



Integration identifier sent to the server on every session for telemetry grouping. Use a short, stable, lowercase slug — e.g. "my-game" or "tryll-roleplay-demo". Leave empty to omit (server defaults to "unknown" when telemetry is enabled). 


        

<hr>



### variable ServerBuildVariant 

```C++
EServerBuildVariant UTryllRuntimeSettings::ServerBuildVariant;
```



Which bundled server build to launch. Only honored when both Default and Release variants are present; otherwise the single present variant is always used. The editor hides this unless both resolve. 


        

<hr>



### variable StorageDataFolder 

```C++
FString UTryllRuntimeSettings::StorageDataFolder;
```



Project-relative folder that node storage paths (string / embedded string storages, voice hotwords) resolve against for this session.


Stored relative to the Unreal project directory in editor builds (e.g. "Content/TryllStringStorages") and relative to the packaged game directory in shipping builds. [**ResolveStorageDataFolderForServer()**](class_u_tryll_runtime_settings.md#function-resolvestoragedatafolderforserver-12) converts this to an absolute path before ConfigureSession sends it on the wire.


Leave empty to use the server's configured storage root. Relative paths in node params resolve against the resolved folder; paths that escape it are rejected. 


        

<hr>



### variable SttEngine 

```C++
ETryllInferenceEngine UTryllRuntimeSettings::SttEngine;
```



Speech-to-text backend. Set to SherpaOnnx to use voice input. 


        

<hr>



### variable TtsEngine 

```C++
ETryllInferenceEngine UTryllRuntimeSettings::TtsEngine;
```



Text-to-speech backend. Set to SherpaOnnx to use spoken replies. 


        

<hr>



### variable bAllowAutoModelDownloading 

```C++
bool UTryllRuntimeSettings::bAllowAutoModelDownloading;
```



When enabled, ConfigureSession tells the server to automatically download any missing models referenced by a graph during CreateAgent, instead of failing immediately. The CreateAgent call will block until the download completes (up to 30 minutes).


DEVELOPMENT AND PROTOTYPING USE ONLY. Disable before shipping to production. In production, use the explicit DownloadModel API with a progress UI. 


        

<hr>



### variable bAutoConfigureSession 

```C++
bool UTryllRuntimeSettings::bAutoConfigureSession;
```



Configure the session automatically once connected, using the engine fields below. Mirrors bAutoLaunchServer, but for session setup: leave on for the simple "it just works" path; turn off to configure the session yourself (e.g. during a loading screen, or to run multiple sessions with different engines). 


        

<hr>



### variable bAutoConnect 

```C++
bool UTryllRuntimeSettings::bAutoConnect;
```



Connect to the server automatically once the subsystem initializes. The built-in connect-retry (ConnectMaxAttempts / ConnectRetryDelaySeconds) covers the brief window while the launched server comes up. Turn off to call Connect() yourself (e.g. from a loading screen). 


        

<hr>



### variable bAutoLaunchServer 

```C++
bool UTryllRuntimeSettings::bAutoLaunchServer;
```



Spawn the bundled tryll\_server.exe on Initialize. Set false to run a server yourself. 


        

<hr>
## Public Functions Documentation




### function GetCategoryName 

```C++
inline virtual FName UTryllRuntimeSettings::GetCategoryName () override const
```




<hr>



### function ResolveStorageDataFolderForServer [1/2]

```C++
inline FString UTryllRuntimeSettings::ResolveStorageDataFolderForServer () const
```



Absolute storage root for ConfigureSession, or empty when StorageDataFolder is unset. 


        

<hr>
## Public Static Functions Documentation




### function GetConfiguredStorageDataFolder 

```C++
static FString UTryllRuntimeSettings::GetConfiguredStorageDataFolder () 
```



Configured StorageDataFolder from DefaultGame.ini (preferred) or the settings CDO. 


        

<hr>



### function GetStorageRootAbsolute 

```C++
static FString UTryllRuntimeSettings::GetStorageRootAbsolute () 
```



Absolute storage root for editor file picking; project dir when unset. 


        

<hr>



### function MakePathRelativeToStorageRoot 

```C++
static bool UTryllRuntimeSettings::MakePathRelativeToStorageRoot (
    const FString & AbsoluteFilePath,
    FString & OutRelativePath
) 
```



Picked absolute file → path relative to the resolved storage root (forward slashes). 


        

<hr>



### function NormalizeStorageRelativePath 

```C++
static FString UTryllRuntimeSettings::NormalizeStorageRelativePath (
    const FString & RelativePath
) 
```



Strips accidental parent-folder prefixes (e.g. "StringStorages/RAG/x" when the storage root is already Content/StringStorages). Paths are relative to storage\_root. 


        

<hr>



### function ResolveStorageDataFolderForServer [2/2]

```C++
static FString UTryllRuntimeSettings::ResolveStorageDataFolderForServer (
    const FString & ConfiguredFolder
) 
```



Project/game-relative configured folder → absolute path for the server. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllRuntimeSettings.h`

