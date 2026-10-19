

# Class ITryllEditorBridge



[**ClassList**](annotated.md) **>** [**ITryllEditorBridge**](class_i_tryll_editor_bridge.md)



[More...](#detailed-description)

* `#include <TryllEditorBridge.h>`





































## Public Functions

| Type | Name |
| ---: | :--- |
| virtual void | [**AcquireConnection**](#function-acquireconnection) () = 0<br> |
| virtual void | [**AdoptRelaunchedServer**](#function-adoptrelaunchedserver) () = 0<br> |
| virtual int32 | [**GetActualPort**](#function-getactualport) () const = 0<br> |
| virtual FTryllConnectionEvents \* | [**GetConnectionEvents**](#function-getconnectionevents) () = 0<br> |
| virtual int64 | [**GetSessionId**](#function-getsessionid) () const = 0<br> |
| virtual FTryllConnection \* | [**GetSharedConnection**](#function-getsharedconnection) () = 0<br> |
| virtual bool | [**HasSessionPauseListener**](#function-hassessionpauselistener) () const = 0<br>_Direct OnPausedNative listeners, or any registered probe that returns true._  |
| virtual bool | [**HasSessionToolCallListener**](#function-hassessiontoolcalllistener) () const = 0<br>_Any registered tool-call probe that returns true._  |
| virtual bool | [**IsConnected**](#function-isconnected) () const = 0<br> |
| virtual bool | [**IsSessionReady**](#function-issessionready) () const = 0<br> |
| virtual int32 | [**RegisterPauseListenerProbe**](#function-registerpauselistenerprobe) (TFunction&lt; bool()&gt; Probe) = 0<br> |
| virtual int32 | [**RegisterToolCallListenerProbe**](#function-registertoolcalllistenerprobe) (TFunction&lt; bool()&gt; Probe) = 0<br> |
| virtual void | [**ReleaseConnection**](#function-releaseconnection) () = 0<br> |
| virtual bool | [**TryLaunch**](#function-trylaunch) (EServerBuildVariant Variant, int32 KnownPort, int32 IdleShutdownTimeoutSeconds, FString & OutError) = 0<br> |
| virtual void | [**UnregisterPauseListenerProbe**](#function-unregisterpauselistenerprobe) (int32 ProbeId) = 0<br> |
| virtual void | [**UnregisterToolCallListenerProbe**](#function-unregistertoolcalllistenerprobe) (int32 ProbeId) = 0<br> |
| virtual  | [**~ITryllEditorBridge**](#function-itrylleditorbridge) () = default<br> |




























## Detailed Description


Runtime-facing surface for the editor-only shared Tryll connection/session. UTryllEditorSubsystem (TryllClientEditor) registers an implementation so [**UTryllSubsystem**](class_u_tryll_subsystem.md) (TryllClient) can share the editor connection in PIE without linking the editor module. Per-message calls are not forwarded: the game subsystem uses [**GetSharedConnection()**](class_i_tryll_editor_bridge.md#function-getsharedconnection) and its own completions. 


    
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



### function GetActualPort 

```C++
virtual int32 ITryllEditorBridge::GetActualPort () const = 0
```




<hr>



### function GetConnectionEvents 

```C++
virtual FTryllConnectionEvents * ITryllEditorBridge::GetConnectionEvents () = 0
```



Broadcast-class events for that connection. Null when there is no connection. 


        

<hr>



### function GetSessionId 

```C++
virtual int64 ITryllEditorBridge::GetSessionId () const = 0
```




<hr>



### function GetSharedConnection 

```C++
virtual FTryllConnection * ITryllEditorBridge::GetSharedConnection () = 0
```



The editor's shared connection. Null when the editor has not connected. 


        

<hr>



### function HasSessionPauseListener 

_Direct OnPausedNative listeners, or any registered probe that returns true._ 
```C++
virtual bool ITryllEditorBridge::HasSessionPauseListener () const = 0
```




<hr>



### function HasSessionToolCallListener 

_Any registered tool-call probe that returns true._ 
```C++
virtual bool ITryllEditorBridge::HasSessionToolCallListener () const = 0
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



Report whether this consumer's session-facing OnPaused is actually bound. Used so a PIE subscription is not mistaken for a listener. Returns an id to pass to UnregisterPauseListenerProbe. 


        

<hr>



### function RegisterToolCallListenerProbe 

```C++
virtual int32 ITryllEditorBridge::RegisterToolCallListenerProbe (
    TFunction< bool()> Probe
) = 0
```



Report whether this consumer's session-facing OnToolCall / OnToolCallWithId / OnToolCallDetailed is actually bound. The native tool-call delegates are the PIE forwarding channels and are always subscribed while delegating, so they cannot tell on their own. A bound listener makes the pause after a tool-call batch resumable (it can answer with ResumeWithToolResult(s)). Returns an id to pass to UnregisterToolCallListenerProbe. 


        

<hr>



### function ReleaseConnection 

```C++
virtual void ITryllEditorBridge::ReleaseConnection () = 0
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



### function UnregisterToolCallListenerProbe 

```C++
virtual void ITryllEditorBridge::UnregisterToolCallListenerProbe (
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

