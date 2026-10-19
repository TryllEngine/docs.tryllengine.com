

# Struct Tryll::Client::TryllClient::SessionConfig



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**TryllClient**](class_tryll_1_1_client_1_1_tryll_client.md) **>** [**SessionConfig**](struct_tryll_1_1_client_1_1_tryll_client_1_1_session_config.md)


























## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::vector&lt; [**EngineSelection**](struct_tryll_1_1_client_1_1_tryll_client_1_1_engine_selection.md) &gt; | [**engines**](#variable-engines)  <br> |
|  std::string | [**gameName**](#variable-gamename)  <br> |
|  std::string | [**locale**](#variable-locale)  <br> |
|  std::vector&lt; [**ModelPlacementHint**](struct_tryll_1_1_client_1_1_tryll_client_1_1_model_placement_hint.md) &gt; | [**placementHints**](#variable-placementhints)  <br>_Optional per-model placement hints._  |
|  std::chrono::milliseconds | [**timeout**](#variable-timeout)   = `std::chrono::seconds{30}`<br> |












































## Public Attributes Documentation




### variable engines 

```C++
std::vector<EngineSelection> Tryll::Client::TryllClient::SessionConfig::engines;
```



Optional, advisory hint: the engines this session expects to touch, as (role, engine) pairs. Leaving it empty is legal and unremarkable — a model's engine comes from the server's catalog, so a load works whether or not this named the engine it needs.


What it buys is timing: the server initialises the named engines' backends at CreateSession instead of at first use, which is worth doing when that cost is large (an ORT CUDA cold start was measured in tens of seconds). A harness that wants determinism about which engines a run touched can also state it here; the editors derive it from their model manifest and expose no engine control.


Only an out-of-range ordinal is rejected (`InvalidEngineSet`, 2005). A duplicate pair is logged and ignored. 


        

<hr>



### variable gameName 

```C++
std::string Tryll::Client::TryllClient::SessionConfig::gameName;
```



Integration identifier for telemetry grouping (e.g. "my-game"). Empty for anonymous sessions. 


        

<hr>



### variable locale 

```C++
std::string Tryll::Client::TryllClient::SessionConfig::locale;
```



Optional session language as a BCP-47 tag ("de", "de-DE", "pt-BR"). One setting the whole voice + text stack derives from: it becomes the default `lang` of every agent voice (VoiceProfile.lang), the language hint for multilingual STT models, `loc/< locale >/` siblings relative storage paths prefer, and the `language` Mustache key templates can write a "reply in X" directive against. Empty = unset (every one of those keeps its pre-locale behaviour). A malformed tag is rejected with `InvalidLocale` (2004). 


        

<hr>



### variable placementHints 

_Optional per-model placement hints._ 
```C++
std::vector<ModelPlacementHint> Tryll::Client::TryllClient::SessionConfig::placementHints;
```




<hr>



### variable timeout 

```C++
std::chrono::milliseconds Tryll::Client::TryllClient::SessionConfig::timeout;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/cpp/include/tryll/TryllClient.h`

