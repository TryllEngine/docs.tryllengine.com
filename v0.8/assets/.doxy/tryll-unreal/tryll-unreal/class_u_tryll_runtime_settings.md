

# Class UTryllRuntimeSettings



[**ClassList**](annotated.md) **>** [**UTryllRuntimeSettings**](class_u_tryll_runtime_settings.md)



[More...](#detailed-description)

* `#include <TryllRuntimeSettings.h>`



Inherits the following classes: UDeveloperSettings


















## Public Attributes

| Type | Name |
| ---: | :--- |
|  int32 | [**ConnectMaxAttempts**](#variable-connectmaxattempts)   = `20`<br> |
|  float | [**ConnectRetryDelaySeconds**](#variable-connectretrydelayseconds)   = `1.0f`<br> |
|  double | [**ConnectionReadyTimeoutSeconds**](#variable-connectionreadytimeoutseconds)   = `10.0`<br> |
|  FString | [**GameName**](#variable-gamename)  <br> |
|  ETryllLlamaCppBackend | [**LlamaCppBackend**](#variable-llamacppbackend)   = `ETryllLlamaCppBackend::Vulkan`<br> |
|  FString | [**Locale**](#variable-locale)   = `TEXT("en")`<br> |
|  EServerBuildVariant | [**ServerBuildVariant**](#variable-serverbuildvariant)   = `EServerBuildVariant::Default`<br> |
|  int32 | [**ServerPort**](#variable-serverport)   = `9100`<br> |
|  double | [**ServerReadinessTimeoutSeconds**](#variable-serverreadinesstimeoutseconds)   = `5.0`<br> |
|  FString | [**StorageDataFolder**](#variable-storagedatafolder)  <br> |
|  bool | [**bAutoConnect**](#variable-bautoconnect)   = `true`<br> |
|  bool | [**bAutoCreateSession**](#variable-bautocreatesession)   = `true`<br> |
|  bool | [**bAutoLaunchServer**](#variable-bautolaunchserver)   = `true`<br> |
|  bool | [**bEnablePlayModeModelGuard**](#variable-benableplaymodemodelguard)   = `true`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
| virtual FName | [**GetCategoryName**](#function-getcategoryname) () override const<br> |
|  TArray&lt; FString &gt; | [**GetLocaleOptions**](#function-getlocaleoptions) () const<br> |
|  FString | [**LlamaCppBackendCliValue**](#function-llamacppbackendclivalue) () const<br> |
|  FString | [**ResolveStorageDataFolderForServer**](#function-resolvestoragedatafolderforserver-12) () const<br> |


## Public Static Functions

| Type | Name |
| ---: | :--- |
|  TArray&lt; TPair&lt; uint8, uint8 &gt; &gt; | [**BuildEngineDeclarations**](#function-buildenginedeclarations) () <br> |
|  TArray&lt; int32 &gt; | [**BuildEngineDeclarationsFlat**](#function-buildenginedeclarationsflat) () <br> |
|  TArray&lt; TPair&lt; FString, uint8 &gt; &gt; | [**BuildHotwordHints**](#function-buildhotwordhints) () <br> |
|  TArray&lt; TPair&lt; FString, uint8 &gt; &gt; | [**BuildPlacementHints**](#function-buildplacementhints) () <br> |
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


Sized as a cold-start backstop: after the launcher hands off a still-warming server (see ServerReadinessTimeoutSeconds), the worker retries connect+hello until the server is ready. ConnectMaxAttempts x (ConnectionReadyTimeoutSeconds + ConnectRetryDelaySeconds) must comfortably exceed a realistic cold start; it then bounds the failure (a clear error) instead of retrying forever. 


        

<hr>



### variable ConnectRetryDelaySeconds 

```C++
float UTryllRuntimeSettings::ConnectRetryDelaySeconds;
```



Constant delay between connect attempts. 


        

<hr>



### variable ConnectionReadyTimeoutSeconds 

```C++
double UTryllRuntimeSettings::ConnectionReadyTimeoutSeconds;
```



Per-attempt bound on how long the connection worker waits for the server's ConnectionReady hello after the TCP connect succeeds. Without this the read was unbounded: a server that accepts the socket but has not finished starting (cold-start "ConnectionReady-late") would hang the worker forever. On timeout the worker reports ConnectFailed, which consumes one attempt and re-drives the connect via the retry budget above. 


        

<hr>



### variable GameName 

```C++
FString UTryllRuntimeSettings::GameName;
```



Integration identifier sent to the server on every session for telemetry grouping. Use a short, stable, lowercase slug — e.g. "my-game" or "tryll-roleplay-demo". Leave empty to omit (server defaults to "unknown" when telemetry is enabled). 


        

<hr>



### variable LlamaCppBackend 

```C++
ETryllLlamaCppBackend UTryllRuntimeSettings::LlamaCppBackend;
```



llama.cpp GPU plugin for an auto-launched server. Vulkan (default) is portable and skips CUDA DLLs in a packaged build. Auto prefers CUDA on NVIDIA when those DLLs were staged. Passed as llama-backend; ignored when bAutoLaunchServer is false. 


        

<hr>



### variable Locale 

```C++
FString UTryllRuntimeSettings::Locale;
```



Session language as a BCP-47 tag ("de", "de-DE", "pt-BR"). One setting the whole voice + text stack derives from, so a language menu does not have to reach into every agent:
* TTS : the default tts\_lang for every GenerateAndSpeak/Speak node (a node that authored its own TtsLang still wins).
* STT : language hint for multilingual recognizers (Whisper, Canary).
* Files : node storage paths prefer loc/&lt;locale&gt;/ siblings when they exist, so localized knowledge bases and canned lines are a folder layout rather than code.
* Text : templates can read the `language` key to write a "reply in X" instruction.




Empty leaves every one of those at its pre-locale behaviour. This is the value used at session start; to change language later call [**UTryllSubsystem::SetSessionLocale**](class_u_tryll_subsystem.md#function-setsessionlocale), which reaches existing agents' voice and template keys but not content already bound to a node. FInternationalization::Get().GetCurrentCulture()-&gt;GetName() is the usual source. 


        

<hr>



### variable ServerBuildVariant 

```C++
EServerBuildVariant UTryllRuntimeSettings::ServerBuildVariant;
```



Which bundled server build to launch. Only honored when both Default and Release variants are present; otherwise the single present variant is always used. The editor hides this unless both resolve. 


        

<hr>



### variable ServerPort 

```C++
int32 UTryllRuntimeSettings::ServerPort;
```



Known port for the shared editor server. Every in-editor launcher (PIE, the chat panel, the model manager panel) probes this port before spawning and reuses whatever it finds there, so one warm server survives across PIE sessions and multiple open tool windows instead of each spawning its own. Packaged/player builds ignore this and always pick a free ephemeral port. 


        

<hr>



### variable ServerReadinessTimeoutSeconds 

```C++
double UTryllRuntimeSettings::ServerReadinessTimeoutSeconds;
```



Seconds the launcher waits _synchronously_ for a freshly spawned server's port to start accepting before it stops blocking the game thread and hands off to the asynchronous connect path. Kept short deliberately: a cold start (DLL paging + first-run antivirus scan of the freshly built exe) can take much longer, and the server is _warming_, not dead — so on timeout the launcher no longer kills it; the connection worker keeps retrying (see ConnectMaxAttempts) until the server is ready. 


        

<hr>



### variable StorageDataFolder 

```C++
FString UTryllRuntimeSettings::StorageDataFolder;
```



Project-relative folder that node storage paths (string / embedded string storages, voice hotwords) resolve against for this session.


Stored relative to the Unreal project directory in editor builds (e.g. "Content/TryllStringStorages") and relative to the packaged game directory in shipping builds. [**ResolveStorageDataFolderForServer()**](class_u_tryll_runtime_settings.md#function-resolvestoragedatafolderforserver-12) converts this to an absolute path before CreateSession sends it on the wire.


Leave empty to use the server's configured storage root. Relative paths in node params resolve against the resolved folder; paths that escape it are rejected. 


        

<hr>



### variable bAutoConnect 

```C++
bool UTryllRuntimeSettings::bAutoConnect;
```



Connect to the server automatically once the subsystem initializes. The built-in connect-retry (ConnectMaxAttempts / ConnectRetryDelaySeconds) covers the brief window while the launched server comes up. Turn off to call Connect() yourself (e.g. from a loading screen). 


        

<hr>



### variable bAutoCreateSession 

```C++
bool UTryllRuntimeSettings::bAutoCreateSession;
```



Configure the session automatically once connected, using the engine fields below. Mirrors bAutoLaunchServer, but for session setup: leave on for the simple "it just works" path; turn off to configure the session yourself (e.g. during a loading screen, or to run multiple sessions with different engines). 


        

<hr>



### variable bAutoLaunchServer 

```C++
bool UTryllRuntimeSettings::bAutoLaunchServer;
```



Spawn the bundled tryll\_server.exe on Initialize. Set false to run a server yourself. 


        

<hr>



### variable bEnablePlayModeModelGuard 

```C++
bool UTryllRuntimeSettings::bEnablePlayModeModelGuard;
```



Editor-only. When true (default), the Play-In-Editor model guard verifies that every registered required model is downloaded before Play starts — it briefly vetoes Play, connects to the server, downloads anything missing (with a progress toast), then resumes Play automatically. Turn this off to enter Play immediately with no pre-flight check (e.g. when you provision models yourself, or to bypass the guard while debugging). No effect in packaged builds, where the guard does not exist. 


        

<hr>
## Public Functions Documentation




### function GetCategoryName 

```C++
inline virtual FName UTryllRuntimeSettings::GetCategoryName () override const
```




<hr>



### function GetLocaleOptions 

```C++
TArray< FString > UTryllRuntimeSettings::GetLocaleOptions () const
```



Choices for the Locale dropdown: the languages the project's _registered_ models declare ([**UTryllModelManifest**](class_u_tryll_model_manifest.md)), never a hardcoded list - so the menu can only ever offer something the project can actually load. Plus an empty entry, because "no session language" is a meaningful choice rather than a missing value.


A **union** across registered models, not an intersection: registering an English-only model for one character must not collapse the whole menu to English. Per-model mismatches surface on the model picker and at agent creation, where they are actionable.


**Fallback, not the primary path.** In the editor, FTryllRuntimeSettingsCustomization replaces this row with a combo that annotates each entry with a catalog model count ("German (de) - 7 models") while still storing the bare code. GetOptions cannot do that: it writes the _selected string_ into the property, so an annotated entry would be stored as the locale. This stays for any surface that renders the property without that customization. 


        

<hr>



### function LlamaCppBackendCliValue 

```C++
inline FString UTryllRuntimeSettings::LlamaCppBackendCliValue () const
```



CLI value for llama-backend ("vulkan" or "auto"). 


        

<hr>



### function ResolveStorageDataFolderForServer [1/2]

```C++
inline FString UTryllRuntimeSettings::ResolveStorageDataFolderForServer () const
```



Absolute storage root for CreateSession, or empty when StorageDataFolder is unset. 


        

<hr>
## Public Static Functions Documentation




### function BuildEngineDeclarations 

```C++
static TArray< TPair< uint8, uint8 > > UTryllRuntimeSettings::BuildEngineDeclarations () 
```



Non-Blueprint form: (ModelRole, InferenceEngine) ordinal pairs. 


        

<hr>



### function BuildEngineDeclarationsFlat 

```C++
static TArray< int32 > UTryllRuntimeSettings::BuildEngineDeclarationsFlat () 
```



The (role, engine) ordinal pairs this project expects to use, derived from the registered models in [**UTryllModelManifest**](class_u_tryll_model_manifest.md) and deduped.


Advisory. It lets the server initialise those engines' backends at CreateSession rather than at first use; a model's engine always comes from the server's catalog, so nothing here gates a load. Empty when nothing is registered, which is legal and is exactly what a fresh project sends.


Derived rather than configured, so it cannot drift from the models the project actually references — and there is deliberately no engine control anywhere in the editor, because which C++ library runs a model is not a question a game developer can answer. 


        

<hr>



### function BuildHotwordHints 

```C++
static TArray< TPair< FString, uint8 > > UTryllRuntimeSettings::BuildHotwordHints () 
```



Per-model hotword-bias hints from the manifest, omitting models left on Default — sparse for the same reason placement is, and because Default and "absent" mean the same thing to the server. 


        

<hr>



### function BuildPlacementHints 

```C++
static TArray< TPair< FString, uint8 > > UTryllRuntimeSettings::BuildPlacementHints () 
```



Per-model placement hints from the manifest, omitting models left on Auto — the hint list is sparse by design. 


        

<hr>



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

