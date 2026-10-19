

# Struct Tryll::Client::AgentProxy::VisemeEvent



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**AgentProxy**](class_tryll_1_1_client_1_1_agent_proxy.md) **>** [**VisemeEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_viseme_event.md)



[More...](#detailed-description)

* `#include <AgentProxy.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::uint32\_t | [**sampleOffset**](#variable-sampleoffset)   = `0`<br> |
|  ::Tryll::WireViseme | [**viseme**](#variable-viseme)   = `::Tryll::WireViseme\_sil`<br> |
|  float | [**weight**](#variable-weight)   = `1.0f`<br> |












































## Detailed Description


One viseme event from a `TtsVisemeFrame`. Timing is turn-absolute at the TTS stream's sample rate. Duration is implied by the next event's offset (or the sentence end). `weight` (0..1) is how far the shape opens, from the loudness of that stretch of speech; closures and rests are always 1.0. 


    
## Public Attributes Documentation




### variable sampleOffset 

```C++
std::uint32_t Tryll::Client::AgentProxy::VisemeEvent::sampleOffset;
```




<hr>



### variable viseme 

```C++
::Tryll::WireViseme Tryll::Client::AgentProxy::VisemeEvent::viseme;
```




<hr>



### variable weight 

```C++
float Tryll::Client::AgentProxy::VisemeEvent::weight;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/cpp/include/tryll/AgentProxy.h`

