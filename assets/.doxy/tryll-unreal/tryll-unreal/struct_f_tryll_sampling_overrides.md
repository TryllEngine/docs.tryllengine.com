

# Struct FTryllSamplingOverrides



[**ClassList**](annotated.md) **>** [**FTryllSamplingOverrides**](struct_f_tryll_sampling_overrides.md)



[More...](#detailed-description)

* `#include <TryllSamplingOverrides.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  float | [**FrequencyPenalty**](#variable-frequencypenalty)   = `0.0`<br> |
|  int32 | [**MaxTokens**](#variable-maxtokens)   = `0`<br> |
|  float | [**MinP**](#variable-minp)   = `0.0`<br> |
|  float | [**PresencePenalty**](#variable-presencepenalty)   = `0.0`<br> |
|  float | [**RepeatPenalty**](#variable-repeatpenalty)   = `0.0`<br> |
|  int32 | [**Seed**](#variable-seed)   = `0`<br> |
|  float | [**Temperature**](#variable-temperature)   = `0.0`<br> |
|  int32 | [**TopK**](#variable-topk)   = `0`<br> |
|  float | [**TopP**](#variable-topp)   = `0.0`<br> |
|  bool | [**bOverrideFrequencyPenalty**](#variable-boverridefrequencypenalty)   = `false`<br> |
|  bool | [**bOverrideMaxTokens**](#variable-boverridemaxtokens)   = `false`<br> |
|  bool | [**bOverrideMinP**](#variable-boverrideminp)   = `false`<br> |
|  bool | [**bOverridePresencePenalty**](#variable-boverridepresencepenalty)   = `false`<br> |
|  bool | [**bOverrideRepeatPenalty**](#variable-boverriderepeatpenalty)   = `false`<br> |
|  bool | [**bOverrideSeed**](#variable-boverrideseed)   = `false`<br> |
|  bool | [**bOverrideTemperature**](#variable-boverridetemperature)   = `false`<br> |
|  bool | [**bOverrideTopK**](#variable-boverridetopk)   = `false`<br> |
|  bool | [**bOverrideTopP**](#variable-boverridetopp)   = `false`<br> |












































## Detailed Description


Sparse sampling override block. Null on any field means "inherit the selected model's default". No per-field (on\_change) attributes: the outer sub-table field carries one callback that re-resolves the whole block and commits derived state. 


    
## Public Attributes Documentation




### variable FrequencyPenalty 

```C++
float FTryllSamplingOverrides::FrequencyPenalty;
```




<hr>



### variable MaxTokens 

```C++
int32 FTryllSamplingOverrides::MaxTokens;
```




<hr>



### variable MinP 

```C++
float FTryllSamplingOverrides::MinP;
```




<hr>



### variable PresencePenalty 

```C++
float FTryllSamplingOverrides::PresencePenalty;
```




<hr>



### variable RepeatPenalty 

```C++
float FTryllSamplingOverrides::RepeatPenalty;
```




<hr>



### variable Seed 

```C++
int32 FTryllSamplingOverrides::Seed;
```




<hr>



### variable Temperature 

```C++
float FTryllSamplingOverrides::Temperature;
```




<hr>



### variable TopK 

```C++
int32 FTryllSamplingOverrides::TopK;
```




<hr>



### variable TopP 

```C++
float FTryllSamplingOverrides::TopP;
```




<hr>



### variable bOverrideFrequencyPenalty 

```C++
bool FTryllSamplingOverrides::bOverrideFrequencyPenalty;
```



OpenAI-style frequency penalty. Null = inherit model default. 


        

<hr>



### variable bOverrideMaxTokens 

```C++
bool FTryllSamplingOverrides::bOverrideMaxTokens;
```



Maximum new tokens to generate. Null = inherit model default. 


        

<hr>



### variable bOverrideMinP 

```C++
bool FTryllSamplingOverrides::bOverrideMinP;
```



Minimum token probability (min-p sampling). Null = inherit model default. 


        

<hr>



### variable bOverridePresencePenalty 

```C++
bool FTryllSamplingOverrides::bOverridePresencePenalty;
```



OpenAI-style presence penalty. Null = inherit model default. 


        

<hr>



### variable bOverrideRepeatPenalty 

```C++
bool FTryllSamplingOverrides::bOverrideRepeatPenalty;
```



Penalise repeated tokens (llama.cpp repeat\_penalty). Null = inherit. 


        

<hr>



### variable bOverrideSeed 

```C++
bool FTryllSamplingOverrides::bOverrideSeed;
```



RNG seed for reproducible sampling. Null = inherit model default. 


        

<hr>



### variable bOverrideTemperature 

```C++
bool FTryllSamplingOverrides::bOverrideTemperature;
```



0.0 = fully deterministic, 2.0 = highly random. Null = inherit. 


        

<hr>



### variable bOverrideTopK 

```C++
bool FTryllSamplingOverrides::bOverrideTopK;
```



Top-k sampling limit. Null = inherit model default. 


        

<hr>



### variable bOverrideTopP 

```C++
bool FTryllSamplingOverrides::bOverrideTopP;
```



Nucleus sampling cutoff. Null = inherit model default. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllSamplingOverrides.h`

