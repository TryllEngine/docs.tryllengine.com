

# Class Tryll::TryllError



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**TryllError**](class_tryll_1_1_tryll_error.md)



_Exception thrown by_ [_**TryllClient**_](class_tryll_1_1_tryll_client.md) _and_[_**AgentProxy**_](class_tryll_1_1_agent_proxy.md) _on server errors or disconnects._[More...](#detailed-description)

* `#include <TryllError.h>`



Inherits the following classes: std::runtime_error


































## Public Functions

| Type | Name |
| ---: | :--- |
|  std::uint32\_t | [**GetCode**](#function-getcode) () noexcept const<br>_Numeric error code from_ `ErrorResponse` _, or 0 for connection-level errors._ |
|  const std::string & | [**GetMessage**](#function-getmessage) () noexcept const<br>_Human-readable error message._  |
|   | [**TryllError**](#function-tryllerror) (std::uint32\_t code, std::string message) <br>_Construct a Tryll client error._  |




























## Detailed Description


The numeric code surfaces the server's `ErrorResponse.code` ranges (1xxx connection, 2xxx session, 3xxx agent, 4xxx model, 5xxx node, 6xxx graph, 7xxx string storage, 8xxx embedded storage). Value 0 is reserved for client-originated failures such as socket errors or timeouts waiting for a response frame. 


    
## Public Functions Documentation




### function GetCode 

_Numeric error code from_ `ErrorResponse` _, or 0 for connection-level errors._
```C++
inline std::uint32_t Tryll::TryllError::GetCode () noexcept const
```



Match against the ranges in `server/common/include/tryll/ErrorCodes.h` to classify the failure family.




**Returns:**

Non-negative code; 0 indicates a client-side failure. 





        

<hr>



### function GetMessage 

_Human-readable error message._ 
```C++
inline const std::string & Tryll::TryllError::GetMessage () noexcept const
```



Identical to the string returned by `what()`, but without the overhead of a C-string view. Safe to reference for the lifetime of the exception object.




**Returns:**

Server or client-produced description of the failure. 





        

<hr>



### function TryllError 

_Construct a Tryll client error._ 
```C++
inline explicit Tryll::TryllError::TryllError (
    std::uint32_t code,
    std::string message
) 
```





**Parameters:**


* `code` Numeric error code from `ErrorCodes.h` ranges, or 0 for client-originated failures. 
* `message` Human-readable description (copied to both the base-class `what()` string and `GetMessage()`). 




        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo/server/client-cpp/include/tryll/TryllError.h`

