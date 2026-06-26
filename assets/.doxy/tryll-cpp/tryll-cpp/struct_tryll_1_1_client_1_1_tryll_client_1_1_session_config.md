

# Struct Tryll::Client::TryllClient::SessionConfig



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**TryllClient**](class_tryll_1_1_client_1_1_tryll_client.md) **>** [**SessionConfig**](struct_tryll_1_1_client_1_1_tryll_client_1_1_session_config.md)



_Per-model-kind session configuration._ [More...](#detailed-description)

* `#include <TryllClient.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  bool | [**allowAutoModelDownloading**](#variable-allowautomodeldownloading)   = `false`<br> |
|  ::Tryll::InferenceEngine | [**embeddingEngine**](#variable-embeddingengine)   = `::Tryll::InferenceEngine\_Mock`<br> |
|  ::Tryll::InferenceEngine | [**engine**](#variable-engine)   = `::Tryll::InferenceEngine\_Mock`<br> |
|  std::string | [**gameName**](#variable-gamename)  <br> |
|  std::string | [**storageDataFolder**](#variable-storagedatafolder)  <br> |
|  ::Tryll::InferenceEngine | [**sttEngine**](#variable-sttengine)   = `::Tryll::InferenceEngine\_Mock`<br> |
|  std::chrono::milliseconds | [**timeout**](#variable-timeout)   = `std::chrono::seconds{30}`<br> |
|  ::Tryll::InferenceEngine | [**ttsEngine**](#variable-ttsengine)   = `::Tryll::InferenceEngine\_Mock`<br> |












































## Detailed Description


Each model kind dispatches against its own engine: language models use engine, STT models use sttEngine, TTS models use ttsEngine, and embedding models use embeddingEngine. All default to ::Tryll::InferenceEngine\_Mock; set the ones you actually use. 


    
## Public Attributes Documentation




### variable allowAutoModelDownloading 

```C++
bool Tryll::Client::TryllClient::SessionConfig::allowAutoModelDownloading;
```



When `true`, the server automatically downloads any missing models referenced by a graph during [**CreateAgent**](class_tryll_1_1_client_1_1_tryll_client.md#function-createagent). Intended for development and prototyping — not for production use. When this flag is set, [**CreateAgent**](class_tryll_1_1_client_1_1_tryll_client.md#function-createagent) and [**CreateAgentAsync**](class_tryll_1_1_client_1_1_tryll_client.md#function-createagentasync) use a 30-minute default timeout unless the caller overrides it explicitly. 


        

<hr>



### variable embeddingEngine 

```C++
::Tryll::InferenceEngine Tryll::Client::TryllClient::SessionConfig::embeddingEngine;
```




<hr>



### variable engine 

```C++
::Tryll::InferenceEngine Tryll::Client::TryllClient::SessionConfig::engine;
```




<hr>



### variable gameName 

```C++
std::string Tryll::Client::TryllClient::SessionConfig::gameName;
```



Integration identifier for telemetry grouping (e.g. "my-game"). Empty for anonymous sessions. 


        

<hr>



### variable storageDataFolder 

```C++
std::string Tryll::Client::TryllClient::SessionConfig::storageDataFolder;
```



Optional per-session storage data folder. Relative storage paths in node params (and voice-input hotwords) resolve against this folder. Empty means not specified — the server falls back to its configured storage\_root (which itself defaults to the executable folder). 


        

<hr>



### variable sttEngine 

```C++
::Tryll::InferenceEngine Tryll::Client::TryllClient::SessionConfig::sttEngine;
```




<hr>



### variable timeout 

```C++
std::chrono::milliseconds Tryll::Client::TryllClient::SessionConfig::timeout;
```




<hr>



### variable ttsEngine 

```C++
::Tryll::InferenceEngine Tryll::Client::TryllClient::SessionConfig::ttsEngine;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/TryllClient.h`

