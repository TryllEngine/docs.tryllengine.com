

# Class UTryllRuntimeSettings



[**ClassList**](annotated.md) **>** [**UTryllRuntimeSettings**](class_u_tryll_runtime_settings.md)



[More...](#detailed-description)

* `#include <TryllRuntimeSettings.h>`



Inherits the following classes: UDeveloperSettings


















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**BuildServerExePath**](#variable-buildserverexepath)   = `TEXT("tryll\_server.exe")`<br> |
|  int32 | [**ConnectMaxAttempts**](#variable-connectmaxattempts)   = `5`<br> |
|  float | [**ConnectRetryDelaySeconds**](#variable-connectretrydelayseconds)   = `1.0f`<br> |
|  FString | [**EditorServerExePath**](#variable-editorserverexepath)  <br> |
|  FString | [**GameName**](#variable-gamename)  <br> |
|  bool | [**bAllowAutoModelDownloading**](#variable-ballowautomodeldownloading)   = `false`<br> |
|  bool | [**bAutoLaunchServer**](#variable-bautolaunchserver)   = `true`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
| virtual FName | [**GetCategoryName**](#function-getcategoryname) () override const<br> |




























## Detailed Description


Project-level settings for the Tryll Client plugin. Edit in Project Settings → Plugins → Tryll Client. Persisted to &lt;YourProject&gt;/Config/DefaultGame.ini under [/Script/TryllClient.TryllRuntimeSettings]. 


    
## Public Attributes Documentation




### variable BuildServerExePath 

```C++
FString UTryllRuntimeSettings::BuildServerExePath;
```



Path to the server exe used in packaged builds. Resolves relative to the game .exe's directory (FPlatformProcess::BaseDir()). 


        

<hr>



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



### variable EditorServerExePath 

```C++
FString UTryllRuntimeSettings::EditorServerExePath;
```



Path to the server exe used in editor builds. Empty disables auto-launch in editor — developers run the server manually. Relative paths resolve against the project dir. 


        

<hr>



### variable GameName 

```C++
FString UTryllRuntimeSettings::GameName;
```



Integration identifier sent to the server on every session for telemetry grouping. Use a short, stable, lowercase slug — e.g. "my-game" or "tryll-roleplay-demo". Leave empty to omit (server defaults to "unknown" when telemetry is enabled). 


        

<hr>



### variable bAllowAutoModelDownloading 

```C++
bool UTryllRuntimeSettings::bAllowAutoModelDownloading;
```



When enabled, ConfigureSession tells the server to automatically download any missing models referenced by a graph during CreateAgent, instead of failing immediately. The CreateAgent call will block until the download completes (up to 30 minutes).


DEVELOPMENT AND PROTOTYPING USE ONLY. Disable before shipping to production. In production, use the explicit DownloadModel API with a progress UI. 


        

<hr>



### variable bAutoLaunchServer 

```C++
bool UTryllRuntimeSettings::bAutoLaunchServer;
```



When true, [**UTryllSubsystem**](class_u_tryll_subsystem.md) spawns tryll\_server.exe automatically on Initialize. 


        

<hr>
## Public Functions Documentation




### function GetCategoryName 

```C++
inline virtual FName UTryllRuntimeSettings::GetCategoryName () override const
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/TryllRuntimeSettings.h`

