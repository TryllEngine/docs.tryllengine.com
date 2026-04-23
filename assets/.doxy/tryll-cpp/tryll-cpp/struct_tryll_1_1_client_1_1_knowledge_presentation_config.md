

# Struct Tryll::Client::KnowledgePresentationConfig



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**KnowledgePresentationConfig**](struct_tryll_1_1_client_1_1_knowledge_presentation_config.md)



_Per-agent knowledge presentation configuration._ [More...](#detailed-description)

* `#include <GraphDescription.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  [**KnowledgeAllEmptyBehavior**](namespace_tryll_1_1_client.md#enum-knowledgeallemptybehavior) | [**allEmptyBehavior**](#variable-allemptybehavior)   = `KnowledgeAllEmptyBehavior::Skip`<br>_Policy for the turn when every retriever returned zero chunks._  |
|  std::string | [**allEmptyTemplate**](#variable-allemptytemplate)  <br>_Alternate template used when_ `allEmptyBehavior` _is_`UseAlternateTemplate` _._ |
|  [**RetrievePresentationConfig**](struct_tryll_1_1_client_1_1_retrieve_presentation_config.md) | [**defaultRetrieve**](#variable-defaultretrieve)  <br>_Default per-source rendering; used for any source without an override._  |
|  std::string | [**messageTemplate**](#variable-messagetemplate)  <br>_Template rendered when at least one retriever has chunks. Must contain_ `{knowledge}` _._ |
|  std::vector&lt; [**RetrievePresentationConfig**](struct_tryll_1_1_client_1_1_retrieve_presentation_config.md) &gt; | [**overrides**](#variable-overrides)  <br>_Per-source rendering overrides keyed by_ `source` _._ |
|  [**KnowledgePlacement**](namespace_tryll_1_1_client.md#enum-knowledgeplacement) | [**placement**](#variable-placement)   = `KnowledgePlacement::InPlaceOfUser`<br>_Placement of the rendered knowledge message._  |












































## Detailed Description


Required on [**Tryll::TryllClient::CreateAgent**](class_tryll_1_1_tryll_client.md#function-createagent) when the graph contains any `Retrieve` node; the server rejects the request otherwise. `messageTemplate` must contain `{knowledge}` and may contain `{user_message}` (only meaningful for `placement` == `InPlaceOfUser`). Unknown template tokens are a validation error returned as `ErrorResponse`. 


    
## Public Attributes Documentation




### variable allEmptyBehavior 

_Policy for the turn when every retriever returned zero chunks._ 
```C++
KnowledgeAllEmptyBehavior Tryll::Client::KnowledgePresentationConfig::allEmptyBehavior;
```




<hr>



### variable allEmptyTemplate 

_Alternate template used when_ `allEmptyBehavior` _is_`UseAlternateTemplate` _._
```C++
std::string Tryll::Client::KnowledgePresentationConfig::allEmptyTemplate;
```




<hr>



### variable defaultRetrieve 

_Default per-source rendering; used for any source without an override._ 
```C++
RetrievePresentationConfig Tryll::Client::KnowledgePresentationConfig::defaultRetrieve;
```




<hr>



### variable messageTemplate 

_Template rendered when at least one retriever has chunks. Must contain_ `{knowledge}` _._
```C++
std::string Tryll::Client::KnowledgePresentationConfig::messageTemplate;
```




<hr>



### variable overrides 

_Per-source rendering overrides keyed by_ `source` _._
```C++
std::vector<RetrievePresentationConfig> Tryll::Client::KnowledgePresentationConfig::overrides;
```




<hr>



### variable placement 

_Placement of the rendered knowledge message._ 
```C++
KnowledgePlacement Tryll::Client::KnowledgePresentationConfig::placement;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/tryll-mono/server/client-cpp/include/tryll/GraphDescription.h`

