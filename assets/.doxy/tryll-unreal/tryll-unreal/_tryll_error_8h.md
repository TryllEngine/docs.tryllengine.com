

# File TryllError.h



[**FileList**](files.md) **>** [**client-unreal**](dir_95d666eee9112f2bfa7f2b736b6243b9.md) **>** [**Source**](dir_e37f8a870dc803113e92bf135247a735.md) **>** [**TryllClient**](dir_0869abba98a308e3c3eadd7e169e0f62.md) **>** [**Public**](dir_338741d27b4bda5805009de80ddaf6fc.md) **>** [**TryllError.h**](_tryll_error_8h.md)





* `#include "CoreMinimal.h"`
* `#include "TryllError.generated.h"`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTryllError**](struct_f_tryll_error.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| enum int32 | [**ETryllErrorCode**](#enum-etryllerrorcode)  <br> |
















































## Public Types Documentation




### enum ETryllErrorCode 

```C++
enum ETryllErrorCode {
    Ok = 0,
    ConnectionFailed = 1001,
    ConnectionDropped = 1002,
    Timeout = 1003,
    ServerShuttingDown = 2001,
    SessionNotReady = 2002,
    InvalidAgentId = 3001,
    AgentAlreadyDestroyed = 3002,
    GraphCompilationFailed = 3003,
    AgentBusy = 3004,
    InferenceFailed = 4001,
    InferenceTimeout = 4002,
    MaxStepsExceeded = 4003,
    MalformedMessage = 5001,
    UnknownMessageType = 5002,
    FrameTooLarge = 5003,
    ProtocolVersionMismatch = 5004,
    DownloadFailed = 6001,
    DownloadNotAvailable = 6002,
    InsufficientDiskSpace = 6003,
    DownloadAlreadyActive = 6004,
    InvalidStringStorageName = 7001,
    StringStorageAlreadyExists = 7002,
    InvalidStringStorageData = 7003
};
```



Numeric error codes sent by the server in ErrorResponse. Mirrors server/common/include/tryll/ErrorCodes.h — values must stay in sync.


Ranges: 1xxx connection-level 2xxx session-level 3xxx agent-level 4xxx inference / workflow 5xxx protocol / framing 6xxx model download 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo/server/client-unreal/Source/TryllClient/Public/TryllError.h`

