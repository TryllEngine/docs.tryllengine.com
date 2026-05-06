

# Struct FTryllError



[**ClassList**](annotated.md) **>** [**FTryllError**](struct_f_tryll_error.md)



[More...](#detailed-description)

* `#include <TryllError.h>`





































## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**FTryllError**](#function-ftryllerror-13) () = default<br> |
|   | [**FTryllError**](#function-ftryllerror-23) (ETryllErrorCode InCode, FString InMessage) <br> |
|   | [**FTryllError**](#function-ftryllerror-33) (uint32 InCode, FString InMessage) <br> |
|  bool | [**IsOk**](#function-isok) () const<br> |
| virtual  | [**UPROPERTY**](#function-uproperty-12) (BlueprintReadOnly, Category="Tryll", meta=(ToolTip="Numeric code matching ETryllErrorCode or a server ErrorCode range. 0 = Ok.")) = 0<br> |
|   | [**UPROPERTY**](#function-uproperty-22) (BlueprintReadOnly, Category="Tryll", meta=(ToolTip="Human-readable error description from the server, or a client-originated message.")) <br> |




























## Detailed Description


Error value returned by Tryll API calls and delivered to Blueprint delegates. 


    
## Public Functions Documentation




### function FTryllError [1/3]

```C++
FTryllError::FTryllError () = default
```




<hr>



### function FTryllError [2/3]

```C++
inline explicit FTryllError::FTryllError (
    ETryllErrorCode InCode,
    FString InMessage
) 
```



Construct from a typed ETryllErrorCode value. 


        

<hr>



### function FTryllError [3/3]

```C++
inline explicit FTryllError::FTryllError (
    uint32 InCode,
    FString InMessage
) 
```



Construct from a raw server-supplied numeric code (e.g. an unknown future value). 


        

<hr>



### function IsOk 

```C++
inline bool FTryllError::IsOk () const
```



True when Code == 0 (no error). Use before reading Message in success-or-failure flows. 


        

<hr>



### function UPROPERTY [1/2]

```C++
virtual FTryllError::UPROPERTY (
    BlueprintReadOnly,
    Category="Tryll",
    meta=(ToolTip="Numeric code matching ETryllErrorCode or a server ErrorCode range. 0 = Ok.")
) = 0
```



Numeric code matching ETryllErrorCode / server ErrorCode range. 


        

<hr>



### function UPROPERTY [2/2]

```C++
FTryllError::UPROPERTY (
    BlueprintReadOnly,
    Category="Tryll",
    meta=(ToolTip="Human-readable error description from the server, or a client-originated message.")
) 
```



Human-readable error description. Safe for display in UI and logs. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo/server/client-unreal/Source/TryllClient/Public/TryllError.h`

