

# Struct Tryll::Client::ManagedServerOptions



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**ManagedServerOptions**](struct_tryll_1_1_client_1_1_managed_server_options.md)



_Configuration passed to_ [_**ManagedServer::Start**_](class_tryll_1_1_client_1_1_managed_server.md#function-start) _._

* `#include <ManagedServer.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::filesystem::path | [**exe**](#variable-exe)  <br> |
|  std::vector&lt; std::string &gt; | [**extraArgs**](#variable-extraargs)  <br>_Extra command-line arguments appended after_ `-- port < port >` _._ |
|  std::string | [**host**](#variable-host)   = `"127.0.0.1"`<br> |
|  std::uint16\_t | [**port**](#variable-port)   = `9100`<br> |
|  std::chrono::milliseconds | [**startTimeout**](#variable-starttimeout)   = `std::chrono::seconds{30}`<br>_How long to wait for the TCP port to become reachable after spawn._  |
|  std::filesystem::path | [**stderrLog**](#variable-stderrlog)  <br> |
|  std::filesystem::path | [**stdoutLog**](#variable-stdoutlog)  <br> |
|  std::chrono::milliseconds | [**stopTimeout**](#variable-stoptimeout)   = `std::chrono::seconds{8}`<br> |
|  std::filesystem::path | [**workingDirectory**](#variable-workingdirectory)  <br> |












































## Public Attributes Documentation




### variable exe 

```C++
std::filesystem::path Tryll::Client::ManagedServerOptions::exe;
```



Path to the `tryll_server` executable. **Required** — no automatic discovery is performed. 


        

<hr>



### variable extraArgs 

_Extra command-line arguments appended after_ `-- port < port >` _._
```C++
std::vector<std::string> Tryll::Client::ManagedServerOptions::extraArgs;
```




<hr>



### variable host 

```C++
std::string Tryll::Client::ManagedServerOptions::host;
```



Host name / IP for the TCP ready-probe after spawn. Almost always `"127.0.0.1"` (the default). 


        

<hr>



### variable port 

```C++
std::uint16_t Tryll::Client::ManagedServerOptions::port;
```



TCP port the server will listen on. Passed as `-- port < port >` on the server command line so it overrides whatever is in `server-config.json`. Default: 9100. 


        

<hr>



### variable startTimeout 

_How long to wait for the TCP port to become reachable after spawn._ 
```C++
std::chrono::milliseconds Tryll::Client::ManagedServerOptions::startTimeout;
```




<hr>



### variable stderrLog 

```C++
std::filesystem::path Tryll::Client::ManagedServerOptions::stderrLog;
```



If non-empty, the child process's stderr is redirected to this file. Empty (default) discards stderr. 


        

<hr>



### variable stdoutLog 

```C++
std::filesystem::path Tryll::Client::ManagedServerOptions::stdoutLog;
```



If non-empty, the child process's stdout is redirected to this file. Empty (default) discards stdout. 


        

<hr>



### variable stopTimeout 

```C++
std::chrono::milliseconds Tryll::Client::ManagedServerOptions::stopTimeout;
```



How long to wait for the process to exit after [**ManagedServer::Stop**](class_tryll_1_1_client_1_1_managed_server.md#function-stop) (or the destructor) sends the termination signal. 


        

<hr>



### variable workingDirectory 

```C++
std::filesystem::path Tryll::Client::ManagedServerOptions::workingDirectory;
```



Working directory for the child process. Empty (default) means the executable's parent directory, which is where the server looks for its `data/` folder by default. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo/server/client-cpp/include/tryll/ManagedServer.h`

