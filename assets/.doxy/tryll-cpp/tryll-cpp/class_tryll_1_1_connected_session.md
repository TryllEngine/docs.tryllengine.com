

# Class Tryll::ConnectedSession



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**ConnectedSession**](class_tryll_1_1_connected_session.md)



_RAII pair: owns a_ [_**Client::ManagedServer**_](class_tryll_1_1_client_1_1_managed_server.md) _child process and a connected_[_**TryllClient**_](class_tryll_1_1_tryll_client.md) _._[More...](#detailed-description)

* `#include <TryllClient.h>`





































## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**ConnectedSession**](#function-connectedsession-13) (ConnectedSession &&) noexcept<br> |
|   | [**ConnectedSession**](#function-connectedsession-23) (const ConnectedSession &) = delete<br> |
|  TryllClient & | [**GetClient**](#function-getclient-12) () noexcept<br>_The connected_ [_**TryllClient**_](class_tryll_1_1_tryll_client.md) _._ |
|  const TryllClient & | [**GetClient**](#function-getclient-22) () noexcept const<br> |
|  [**Client::ManagedServer**](class_tryll_1_1_client_1_1_managed_server.md) & | [**GetServer**](#function-getserver-12) () noexcept<br>_The running_ [_**Client::ManagedServer**_](class_tryll_1_1_client_1_1_managed_server.md) _._ |
|  const [**Client::ManagedServer**](class_tryll_1_1_client_1_1_managed_server.md) & | [**GetServer**](#function-getserver-22) () noexcept const<br> |
|  void | [**Shutdown**](#function-shutdown) (std::chrono::milliseconds timeout=std::chrono::seconds{30}) <br>_Shut the client down, then stop the server. Idempotent._  |
|  ConnectedSession & | [**operator=**](#function-operator) (ConnectedSession &&) noexcept<br> |
|  ConnectedSession & | [**operator=**](#function-operator_1) (const ConnectedSession &) = delete<br> |
|   | [**~ConnectedSession**](#function-connectedsession) () <br> |




























## Detailed Description


Move-only. On destruction, the [**TryllClient**](class_tryll_1_1_tryll_client.md) is shut down first (graceful TCP close), then the [**Client::ManagedServer**](class_tryll_1_1_client_1_1_managed_server.md) is stopped (process termination). The declaration order of the members enforces this.


Obtain a [**ConnectedSession**](class_tryll_1_1_connected_session.md) via [**TryllClient::RunAndConnect**](class_tryll_1_1_tryll_client.md#function-runandconnect).



```C++
auto session = Tryll::TryllClient::RunAndConnect(opts);
session.GetClient().ConfigureSession(Tryll::Client::InferenceEngine::LlamaCpp);
// …
```
 


    
## Public Functions Documentation




### function ConnectedSession [1/3]

```C++
Tryll::ConnectedSession::ConnectedSession (
    ConnectedSession &&
) noexcept
```




<hr>



### function ConnectedSession [2/3]

```C++
Tryll::ConnectedSession::ConnectedSession (
    const ConnectedSession &
) = delete
```




<hr>



### function GetClient [1/2]

_The connected_ [_**TryllClient**_](class_tryll_1_1_tryll_client.md) _._
```C++
TryllClient & Tryll::ConnectedSession::GetClient () noexcept
```




<hr>



### function GetClient [2/2]

```C++
const TryllClient & Tryll::ConnectedSession::GetClient () noexcept const
```




<hr>



### function GetServer [1/2]

_The running_ [_**Client::ManagedServer**_](class_tryll_1_1_client_1_1_managed_server.md) _._
```C++
Client::ManagedServer & Tryll::ConnectedSession::GetServer () noexcept
```




<hr>



### function GetServer [2/2]

```C++
const Client::ManagedServer & Tryll::ConnectedSession::GetServer () noexcept const
```




<hr>



### function Shutdown 

_Shut the client down, then stop the server. Idempotent._ 
```C++
void Tryll::ConnectedSession::Shutdown (
    std::chrono::milliseconds timeout=std::chrono::seconds{30}
) 
```





**Parameters:**


* `timeout` Maximum time to wait for a graceful client shutdown. 




        

<hr>



### function operator= 

```C++
ConnectedSession & Tryll::ConnectedSession::operator= (
    ConnectedSession &&
) noexcept
```




<hr>



### function operator= 

```C++
ConnectedSession & Tryll::ConnectedSession::operator= (
    const ConnectedSession &
) = delete
```




<hr>



### function ~ConnectedSession 

```C++
Tryll::ConnectedSession::~ConnectedSession () 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo/server/client-cpp/include/tryll/TryllClient.h`

