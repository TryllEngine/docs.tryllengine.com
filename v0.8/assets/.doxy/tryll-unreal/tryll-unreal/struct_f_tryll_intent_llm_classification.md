

# Struct FTryllIntentLlmClassification



[**ClassList**](annotated.md) **>** [**FTryllIntentLlmClassification**](struct_f_tryll_intent_llm_classification.md)



[More...](#detailed-description)

* `#include <TryllIntentClassification.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  int64 | [**AgentId**](#variable-agentid)   = `0`<br> |
|  FString | [**Intent**](#variable-intent)  <br> |
|  TArray&lt; [**FTryllIntentLlmLabelProb**](struct_f_tryll_intent_llm_label_prob.md) &gt; | [**LabelProbs**](#variable-labelprobs)  <br> |
|  FString | [**NodeName**](#variable-nodename)  <br> |
|  FString | [**NotFoundReason**](#variable-notfoundreason)  <br> |
|  float | [**SecondProb**](#variable-secondprob)   = `0.f`<br> |
|  float | [**TopProb**](#variable-topprob)   = `0.f`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  bool | [**Accepted**](#function-accepted) () const<br> |




























## Detailed Description


ClassifyIntentLLM payload (event\_type="intent\_llm\_classified"). 


    
## Public Attributes Documentation




### variable AgentId 

```C++
int64 FTryllIntentLlmClassification::AgentId;
```




<hr>



### variable Intent 

```C++
FString FTryllIntentLlmClassification::Intent;
```




<hr>



### variable LabelProbs 

```C++
TArray<FTryllIntentLlmLabelProb> FTryllIntentLlmClassification::LabelProbs;
```




<hr>



### variable NodeName 

```C++
FString FTryllIntentLlmClassification::NodeName;
```




<hr>



### variable NotFoundReason 

```C++
FString FTryllIntentLlmClassification::NotFoundReason;
```




<hr>



### variable SecondProb 

```C++
float FTryllIntentLlmClassification::SecondProb;
```




<hr>



### variable TopProb 

```C++
float FTryllIntentLlmClassification::TopProb;
```




<hr>
## Public Functions Documentation




### function Accepted 

```C++
inline bool FTryllIntentLlmClassification::Accepted () const
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllIntentClassification.h`

