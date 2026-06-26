

# Class Tryll::Client::ManagedServer



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**ManagedServer**](class_tryll_1_1_client_1_1_managed_server.md)



_RAII handle around a child_ `tryll_server` _process._[More...](#detailed-description)

* `#include <ManagedServer.h>`





































## Public Functions

| Type | Name |
| ---: | :--- |
|  std::string | [**Host**](#function-host) () noexcept const<br>_The host string from_ [_**ManagedServerOptions::host**_](struct_tryll_1_1_client_1_1_managed_server_options.md#variable-host) _._ |
|  bool | [**IsRunning**](#function-isrunning) () noexcept const<br>_Returns_ `true` _while the child process is still running._ |
|   | [**ManagedServer**](#function-managedserver-13) (ManagedServer &&) noexcept<br> |
|   | [**ManagedServer**](#function-managedserver-23) (const ManagedServer &) = delete<br> |
|  std::uint16\_t | [**Port**](#function-port) () noexcept const<br>_The port from_ [_**ManagedServerOptions::port**_](struct_tryll_1_1_client_1_1_managed_server_options.md#variable-port) _._ |
|  void | [**Stop**](#function-stop) (std::chrono::milliseconds timeout=std::chrono::seconds{8}) <br>_Terminate the child process._  |
|  ManagedServer & | [**operator=**](#function-operator) (ManagedServer &&) noexcept<br> |
|  ManagedServer & | [**operator=**](#function-operator_1) (const ManagedServer &) = delete<br> |
|   | [**~ManagedServer**](#function-managedserver) () <br> |


## Public Static Functions

| Type | Name |
| ---: | :--- |
|  ManagedServer | [**Start**](#function-start) (const [**ManagedServerOptions**](struct_tryll_1_1_client_1_1_managed_server_options.md) & opts) <br>_Spawn the server and block until its TCP port accepts connections._  |


























## Detailed Description


Move-only. The destructor calls [**Stop**](class_tryll_1_1_client_1_1_managed_server.md#function-stop) if the process is still running. 


    
## Public Functions Documentation




### function Host 

_The host string from_ [_**ManagedServerOptions::host**_](struct_tryll_1_1_client_1_1_managed_server_options.md#variable-host) _._
```C++
std::string Tryll::Client::ManagedServer::Host () noexcept const
```




<hr>



### function IsRunning 

_Returns_ `true` _while the child process is still running._
```C++
bool Tryll::Client::ManagedServer::IsRunning () noexcept const
```




<hr>



### function ManagedServer [1/3]

```C++
Tryll::Client::ManagedServer::ManagedServer (
    ManagedServer &&
) noexcept
```




<hr>



### function ManagedServer [2/3]

```C++
Tryll::Client::ManagedServer::ManagedServer (
    const ManagedServer &
) = delete
```




<hr>



### function Port 

_The port from_ [_**ManagedServerOptions::port**_](struct_tryll_1_1_client_1_1_managed_server_options.md#variable-port) _._
```C++
std::uint16_t Tryll::Client::ManagedServer::Port () noexcept const
```




<hr>



### function Stop 

_Terminate the child process._ 
```C++
void Tryll::Client::ManagedServer::Stop (
    std::chrono::milliseconds timeout=std::chrono::seconds{8}
) 
```



Sends a termination signal and waits up to `timeout` for the process to exit. If the process does not exit in time it is force-killed. Idempotent — safe to call more than once.




**Parameters:**


* `timeout` Override for the stop timeout set in the options. 




        

<hr>



### function operator= 

```C++
ManagedServer & Tryll::Client::ManagedServer::operator= (
    ManagedServer &&
) noexcept
```




<hr>



### function operator= 

```C++
ManagedServer & Tryll::Client::ManagedServer::operator= (
    const ManagedServer &
) = delete
```




<hr>



### function ~ManagedServer 

```C++
Tryll::Client::ManagedServer::~ManagedServer () 
```




<hr>
## Public Static Functions Documentation




### function Start 

_Spawn the server and block until its TCP port accepts connections._ 
```C++
static ManagedServer Tryll::Client::ManagedServer::Start (
    const ManagedServerOptions & opts
) 
```



The server is launched with `--port <opts.port>` so the port is authoritative from the caller's side regardless of `server-config.json`.




**Parameters:**


* `opts` Launch options. `opts.exe` must exist.



**Returns:**

A ready-to-use [**ManagedServer**](class_tryll_1_1_client_1_1_managed_server.md).




**Exception:**


* `TryllError` If the executable is not found, the process fails to start, the process exits before the port opens, or `opts.startTimeout` expires. 




        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/ManagedServer.h`

