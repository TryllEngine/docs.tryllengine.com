

# File TryllError.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllError.h**](_tryll_error_8h.md)





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
    InvalidStringStorageData = 7003,
    SttModelLoadFailed = 4100,
    AudioFormatUnsupported = 4101,
    VoiceInputNotFound = 4102,
    UtteranceInProgress = 4103,
    NoActiveUtterance = 4104,
    UtteranceTimeout = 4105
};
```



Numeric error codes sent by the server in ErrorResponse. Mirrors server/common/include/tryll/ErrorCodes.h — values must stay in sync.


Ranges: 1xxx connection-level 2xxx session-level 3xxx agent-level 4xxx inference / workflow 5xxx protocol / framing 6xxx model download 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllError.h`

