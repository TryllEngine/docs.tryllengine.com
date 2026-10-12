

# Class FTryllManagedServerProcess



[**ClassList**](annotated.md) **>** [**FTryllManagedServerProcess**](class_f_tryll_managed_server_process.md)



[More...](#detailed-description)

* `#include <TryllManagedServerProcess.h>`

















## Public Types

| Type | Name |
| ---: | :--- |
| enum uint8 | [**ELineSeverity**](#enum-elineseverity)  <br> |






## Public Static Attributes

| Type | Name |
| ---: | :--- |
|  const TCHAR \*const | [**ClientDevMode**](#variable-clientdevmode)  <br> |
|  const TCHAR \*const | [**ServerModeEnvVar**](#variable-servermodeenvvar)  <br> |
|  const TCHAR \*const | [**TryllDevMode**](#variable-trylldevmode)  <br> |














## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**FTryllManagedServerProcess**](#function-ftryllmanagedserverprocess-12) () = default<br> |
|   | [**FTryllManagedServerProcess**](#function-ftryllmanagedserverprocess-22) (const [**FTryllManagedServerProcess**](class_f_tryll_managed_server_process.md) &) = delete<br> |
|  bool | [**IsObservedReady**](#function-isobservedready) () const<br> |
|  bool | [**IsRunning**](#function-isrunning) () <br> |
|  void | [**ReleaseHandle**](#function-releasehandle) () <br> |
|  bool | [**Spawn**](#function-spawn) (const FString & ExePath, int32 Port, int32 IdleShutdownTimeoutSeconds, double ReadinessTimeoutSeconds, const FString & LlamaCppBackend, FString & OutError, bool & bOutPortConflict) <br> |
|  void | [**TerminateAndRelease**](#function-terminateandrelease) () <br> |
|  [**FTryllManagedServerProcess**](class_f_tryll_managed_server_process.md) & | [**operator=**](#function-operator) (const [**FTryllManagedServerProcess**](class_f_tryll_managed_server_process.md) &) = delete<br> |
|   | [**~FTryllManagedServerProcess**](#function-ftryllmanagedserverprocess) () <br> |


## Public Static Functions

| Type | Name |
| ---: | :--- |
|  [**ELineSeverity**](class_f_tryll_managed_server_process.md#enum-elineseverity) | [**ClassifyLine**](#function-classifyline) (const FString & Line) <br> |
|  const FString & | [**GetDeveloperModeOverride**](#function-getdevelopermodeoverride) () <br> |
|  FString | [**NormalizeLaunchMode**](#function-normalizelaunchmode) (const FString & Value) <br> |
|  FString | [**ResolveLaunchMode**](#function-resolvelaunchmode) () <br> |
|  void | [**SetDeveloperModeOverride**](#function-setdevelopermodeoverride) (const FString & Mode) <br> |


























## Detailed Description


Plain (non-UObject) helper that spawns tryll\_server.exe as a managed child, draining its stdout to the log until released. Shared by UTryllEditorSubsystem and [**UTryllSubsystem**](class_u_tryll_subsystem.md)'s packaged-build path — those differ in when/which port to spawn on, not in how. 


    
## Public Types Documentation




### enum ELineSeverity 

```C++
enum FTryllManagedServerProcess::ELineSeverity {
    Normal,
    Warning,
    Error
};
```



Severity of one server stdout line, read from its `[time] [level] message` prefix. 


        

<hr>
## Public Static Attributes Documentation




### variable ClientDevMode 

```C++
const TCHAR* const FTryllManagedServerProcess::ClientDevMode;
```




<hr>



### variable ServerModeEnvVar 

```C++
const TCHAR* const FTryllManagedServerProcess::ServerModeEnvVar;
```



Environment variable: "tryll-dev" or "client-dev". Wins over the developer override. 


        

<hr>



### variable TryllDevMode 

```C++
const TCHAR* const FTryllManagedServerProcess::TryllDevMode;
```




<hr>
## Public Functions Documentation




### function FTryllManagedServerProcess [1/2]

```C++
FTryllManagedServerProcess::FTryllManagedServerProcess () = default
```




<hr>



### function FTryllManagedServerProcess [2/2]

```C++
FTryllManagedServerProcess::FTryllManagedServerProcess (
    const FTryllManagedServerProcess &
) = delete
```




<hr>



### function IsObservedReady 

```C++
inline bool FTryllManagedServerProcess::IsObservedReady () const
```



True after Spawn saw the child accept TCP. False while still warming. 


        

<hr>



### function IsRunning 

```C++
bool FTryllManagedServerProcess::IsRunning () 
```




<hr>



### function ReleaseHandle 

```C++
void FTryllManagedServerProcess::ReleaseHandle () 
```



Releases the handle without killing the process (normal teardown). 


        

<hr>



### function Spawn 

```C++
bool FTryllManagedServerProcess::Spawn (
    const FString & ExePath,
    int32 Port,
    int32 IdleShutdownTimeoutSeconds,
    double ReadinessTimeoutSeconds,
    const FString & LlamaCppBackend,
    FString & OutError,
    bool & bOutPortConflict
) 
```



Spawns ExePath with "--port Port --idle-shutdown-timeout Seconds
--llama-backend LlamaCppBackend" and waits up to ReadinessTimeoutSeconds for its TCP port to start accepting.


Returns true when the launch should proceed to connecting — either the port is already accepting, OR the readiness wait elapsed but the process is _still alive_ (a cold start still warming up). In the latter case the server is deliberately left running, not killed: killing an almost-ready server discards the warmup and forces a cold re-spawn — the root cause of "first launch hangs, relaunch works". The caller connects anyway and the connection worker retries until the server is ready.


Returns false only for a genuine launch failure: the process could not be spawned, or it exited on its own before the port opened. OutError is then populated with a clear reason, and bOutPortConflict is set to true iff the process exited with Tryll::ServerExitPortInUse (3), so callers can special-case a bind collision (retry on a different port). 


        

<hr>



### function TerminateAndRelease 

```C++
void FTryllManagedServerProcess::TerminateAndRelease () 
```



Force-terminate then release (test/CI cleanup only). 


        

<hr>



### function operator= 

```C++
FTryllManagedServerProcess & FTryllManagedServerProcess::operator= (
    const FTryllManagedServerProcess &
) = delete
```




<hr>



### function ~FTryllManagedServerProcess 

```C++
FTryllManagedServerProcess::~FTryllManagedServerProcess () 
```




<hr>
## Public Static Functions Documentation




### function ClassifyLine 

```C++
static ELineSeverity FTryllManagedServerProcess::ClassifyLine (
    const FString & Line
) 
```




<hr>



### function GetDeveloperModeOverride 

```C++
static inline const FString & FTryllManagedServerProcess::GetDeveloperModeOverride () 
```




<hr>



### function NormalizeLaunchMode 

```C++
static FString FTryllManagedServerProcess::NormalizeLaunchMode (
    const FString & Value
) 
```



Canonical mode for Value, or empty when it is not a mode a plugin may launch. 


        

<hr>



### function ResolveLaunchMode 

```C++
static FString FTryllManagedServerProcess::ResolveLaunchMode () 
```



Mode for the next spawn: ServerModeEnvVar, then the developer override, then client-dev. tryll-dev spawns also pass disable-telemetry. 


        

<hr>



### function SetDeveloperModeOverride 

```C++
static void FTryllManagedServerProcess::SetDeveloperModeOverride (
    const FString & Mode
) 
```



Set by the monorepo-only editor developer menu (compiled only when TRYLL\_DEVELOPER\_TOOLS=1). Empty clears it. Released plugins never call this. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllManagedServerProcess.h`

