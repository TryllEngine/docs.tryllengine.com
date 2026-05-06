

# File TryllAgentComponent.h



[**FileList**](files.md) **>** [**client-unreal**](dir_95d666eee9112f2bfa7f2b736b6243b9.md) **>** [**Source**](dir_e37f8a870dc803113e92bf135247a735.md) **>** [**TryllClient**](dir_0869abba98a308e3c3eadd7e169e0f62.md) **>** [**Public**](dir_338741d27b4bda5805009de80ddaf6fc.md) **>** [**TryllAgentComponent.h**](_tryll_agent_component_8h.md)





* `#include "CoreMinimal.h"`
* `#include "Components/ActorComponent.h"`
* `#include "TryllGraphDescription.h"`
* `#include "TryllError.h"`
* `#include "TryllAgentComponent.generated.h"`





































## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE**](#function-declare_dynamic_multicast_delegate) (FOnTryllAgentReady) <br> |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_OneParam**](#function-declare_dynamic_multicast_delegate_oneparam) (FOnTryllAgentError, const FString &, ErrorMessage) <br> |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_OneParam**](#function-declare_dynamic_multicast_delegate_oneparam) (FOnTryllAgentAnswerFull, const FString &, FullText) <br> |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_ThreeParams**](#function-declare_dynamic_multicast_delegate_threeparams) (FOnTryllAgentTurnComplete, ETryllTurnStatus, Status, const FString &, DebugInfo, int32, TokensGenerated) <br> |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_dynamic_multicast_delegate_twoparams) (FOnTryllAgentAnswerText, const FString &, Text, bool, bIsFinal) <br> |
|   | [**DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_TwoParams**](#function-declare_dynamic_multicast_delegate_twoparams) (FOnTryllParamChanged, bool, bSuccess, const FString &, ErrorMessage) <br> |




























## Public Functions Documentation




### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE (
    FOnTryllAgentReady
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_OneParam 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam (
    FOnTryllAgentError,
    const FString &,
    ErrorMessage
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_OneParam 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam (
    FOnTryllAgentAnswerFull,
    const FString &,
    FullText
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_ThreeParams 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParams (
    FOnTryllAgentTurnComplete,
    ETryllTurnStatus,
    Status,
    const FString &,
    DebugInfo,
    int32,
    TokensGenerated
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams (
    FOnTryllAgentAnswerText,
    const FString &,
    Text,
    bool,
    bIsFinal
) 
```




<hr>



### function DECLARE\_DYNAMIC\_MULTICAST\_DELEGATE\_TwoParams 

```C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams (
    FOnTryllParamChanged,
    bool,
    bSuccess,
    const FString &,
    ErrorMessage
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo/server/client-unreal/Source/TryllClient/Public/TryllAgentComponent.h`

