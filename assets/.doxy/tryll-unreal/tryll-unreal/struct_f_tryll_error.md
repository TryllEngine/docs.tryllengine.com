

# Struct FTryllError



[**ClassList**](annotated.md) **>** [**FTryllError**](struct_f_tryll_error.md)



[More...](#detailed-description)

* `#include <TryllError.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  int32 | [**Code**](#variable-code)   = `0`<br> |
|  FString | [**Message**](#variable-message)  <br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**FTryllError**](#function-ftryllerror-13) () = default<br> |
|   | [**FTryllError**](#function-ftryllerror-23) (ETryllErrorCode InCode, FString InMessage) <br> |
|   | [**FTryllError**](#function-ftryllerror-33) (uint32 InCode, FString InMessage) <br> |
|  bool | [**IsOk**](#function-isok) () const<br> |




























## Detailed Description


Error value returned by Tryll API calls and delivered to Blueprint delegates. 


    
## Public Attributes Documentation




### variable Code 

```C++
int32 FTryllError::Code;
```



Numeric code matching ETryllErrorCode / server ErrorCode range. 


        

<hr>



### variable Message 

```C++
FString FTryllError::Message;
```



Human-readable error description. Safe for display in UI and logs. 


        

<hr>
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

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllError.h`

