

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
|  bool | [**Spawn**](#function-spawn) (const FString & ExePath, int32 Port, int32 IdleShutdownTimeoutSeconds, FString & OutError, bool & bOutPortConflict) <br> |
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
    FString & OutError,
    bool & bOutPortConflict
) 
```



Spawns ExePath with "--port Port --idle-shutdown-timeout Seconds" and blocks until its TCP port accepts a connection or a bounded readiness timeout elapses — without this, callers would report launch success as soon as the OS process handle exists, even though cold startup (config/model-catalog loading) or a port collision can leave the port unreachable, or unreachable forever if the process already exited.


On failure, OutError is populated with a clear reason. bOutPortConflict is set to true iff the process exited with Tryll::ServerExitPortInUse (3), so callers can special-case a bind collision (retry on a different port) versus any other failure. 


        

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

