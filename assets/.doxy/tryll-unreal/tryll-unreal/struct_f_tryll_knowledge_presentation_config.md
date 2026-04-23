

# Struct FTryllKnowledgePresentationConfig



[**ClassList**](annotated.md) **>** [**FTryllKnowledgePresentationConfig**](struct_f_tryll_knowledge_presentation_config.md)



[More...](#detailed-description)

* `#include <TryllGraphDescription.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  ETryllKnowledgeAllEmptyBehavior | [**AllEmptyBehavior**](#variable-allemptybehavior)   = `ETryllKnowledgeAllEmptyBehavior::Skip`<br> |
|  FString | [**AllEmptyTemplate**](#variable-allemptytemplate)  <br> |
|  [**FTryllRetrievePresentationConfig**](struct_f_tryll_retrieve_presentation_config.md) | [**DefaultRetrieve**](#variable-defaultretrieve)  <br> |
|  FString | [**MessageTemplate**](#variable-messagetemplate)  <br> |
|  TArray&lt; [**FTryllRetrievePresentationConfig**](struct_f_tryll_retrieve_presentation_config.md) &gt; | [**Overrides**](#variable-overrides)  <br> |
|  ETryllKnowledgePlacement | [**Placement**](#variable-placement)   = `ETryllKnowledgePlacement::InPlaceOfUser`<br> |












































## Detailed Description


Per-agent knowledge presentation configuration. Required when the graph has a Retrieve node. 


    
## Public Attributes Documentation




### variable AllEmptyBehavior 

```C++
ETryllKnowledgeAllEmptyBehavior FTryllKnowledgePresentationConfig::AllEmptyBehavior;
```




<hr>



### variable AllEmptyTemplate 

```C++
FString FTryllKnowledgePresentationConfig::AllEmptyTemplate;
```



Required when AllEmptyBehavior == UseAlternateTemplate. 


        

<hr>



### variable DefaultRetrieve 

```C++
FTryllRetrievePresentationConfig FTryllKnowledgePresentationConfig::DefaultRetrieve;
```




<hr>



### variable MessageTemplate 

```C++
FString FTryllKnowledgePresentationConfig::MessageTemplate;
```



Template for the knowledge message. Must contain {knowledge}. May also contain {user\_message} (only meaningful for InPlaceOfUser placement). 


        

<hr>



### variable Overrides 

```C++
TArray<FTryllRetrievePresentationConfig> FTryllKnowledgePresentationConfig::Overrides;
```




<hr>



### variable Placement 

```C++
ETryllKnowledgePlacement FTryllKnowledgePresentationConfig::Placement;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/tryll-mono/server/client-unreal/Source/TryllClient/Public/TryllGraphDescription.h`

