

# Class FTryllManagedServerProcess



[**ClassList**](annotated.md) **>** [**FTryllManagedServerProcess**](class_f_tryll_managed_server_process.md)



[More...](#detailed-description)

* `#include <TryllManagedServerProcess.h>`





































## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**FTryllManagedServerProcess**](#function-ftryllmanagedserverprocess-12) () = default<br> |
|   | [**FTryllManagedServerProcess**](#function-ftryllmanagedserverprocess-22) (const [**FTryllManagedServerProcess**](class_f_tryll_managed_server_process.md) &) = delete<br> |
|  bool | [**IsRunning**](#function-isrunning) () <br> |
|  void | [**ReleaseHandle**](#function-releasehandle) () <br> |
|  bool | [**Spawn**](#function-spawn) (const FString & ExePath, int32 Port, int32 IdleShutdownTimeoutSeconds, double ReadinessTimeoutSeconds, FString & OutError, bool & bOutPortConflict) <br> |
|  void | [**TerminateAndRelease**](#function-terminateandrelease) () <br> |
|  [**FTryllManagedServerProcess**](class_f_tryll_managed_server_process.md) & | [**operator=**](#function-operator) (const [**FTryllManagedServerProcess**](class_f_tryll_managed_server_process.md) &) = delete<br> |
|   | [**~FTryllManagedServerProcess**](#function-ftryllmanagedserverprocess) () <br> |




























## Detailed Description


Plain (non-UObject) helper that spawns tryll\_server.exe as a managed child, draining its stdout to the log until released. Shared by UTryllEditorSubsystem and [**UTryllSubsystem**](class_u_tryll_subsystem.md)'s packaged-build path — those differ in when/which port to spawn on, not in how. 


    
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
    FString & OutError,
    bool & bOutPortConflict
) 
```



Spawns ExePath with "--port Port --idle-shutdown-timeout Seconds" and waits up to ReadinessTimeoutSeconds for its TCP port to start accepting.


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

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllManagedServerProcess.h`

