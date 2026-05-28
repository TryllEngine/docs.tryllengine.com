

# Struct FTryllSamplingOverrides



[**ClassList**](annotated.md) **>** [**FTryllSamplingOverrides**](struct_f_tryll_sampling_overrides.md)



[More...](#detailed-description)

* `#include <TryllSamplingOverrides.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  bool | [**bOverrideFrequencyPenalty**](#variable-boverridefrequencypenalty)   = `false`<br> |
|  bool | [**bOverrideMaxTokens**](#variable-boverridemaxtokens)   = `false`<br> |
|  bool | [**bOverrideMinP**](#variable-boverrideminp)   = `false`<br> |
|  bool | [**bOverridePresencePenalty**](#variable-boverridepresencepenalty)   = `false`<br> |
|  bool | [**bOverrideRepeatPenalty**](#variable-boverriderepeatpenalty)   = `false`<br> |
|  bool | [**bOverrideSeed**](#variable-boverrideseed)   = `false`<br> |
|  bool | [**bOverrideTemperature**](#variable-boverridetemperature)   = `false`<br> |
|  bool | [**bOverrideTopK**](#variable-boverridetopk)   = `false`<br> |
|  bool | [**bOverrideTopP**](#variable-boverridetopp)   = `false`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
| virtual  | [**UPROPERTY**](#function-uproperty-19) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|SamplingOverrides", meta=(EditCondition="bOverrideTemperature", ClampMin="0.0", ClampMax="2.0", UIMin="0.0", UIMax="2.0")) = 0<br> |
| virtual  | [**UPROPERTY**](#function-uproperty-29) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|SamplingOverrides", meta=(EditCondition="bOverrideTopP", ClampMin="0.0", ClampMax="1.0", UIMin="0.0", UIMax="1.0")) = 0<br> |
| virtual  | [**UPROPERTY**](#function-uproperty-39) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|SamplingOverrides", meta=(EditCondition="bOverrideTopK", ClampMin="0.0", UIMin="0.0")) = 0<br> |
| virtual  | [**UPROPERTY**](#function-uproperty-49) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|SamplingOverrides", meta=(EditCondition="bOverrideMinP", ClampMin="0.0", ClampMax="1.0", UIMin="0.0", UIMax="1.0")) = 0<br> |
| virtual  | [**UPROPERTY**](#function-uproperty-59) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|SamplingOverrides", meta=(EditCondition="bOverrideMaxTokens", ClampMin="1.0", UIMin="1.0")) = 0<br> |
| virtual  | [**UPROPERTY**](#function-uproperty-69) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|SamplingOverrides", meta=(EditCondition="bOverrideSeed")) = 0<br> |
| virtual  | [**UPROPERTY**](#function-uproperty-79) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|SamplingOverrides", meta=(EditCondition="bOverrideRepeatPenalty", ClampMin="0.0", UIMin="0.0")) = 0<br> |
| virtual  | [**UPROPERTY**](#function-uproperty-89) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|SamplingOverrides", meta=(EditCondition="bOverridePresencePenalty")) = 0<br> |
| virtual  | [**UPROPERTY**](#function-uproperty-99) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|SamplingOverrides", meta=(EditCondition="bOverrideFrequencyPenalty")) = 0<br> |




























## Detailed Description


Sparse sampling override block. Null on any field means "inherit the selected model's default". No per-field (on\_change) attributes: the outer sub-table field carries one callback that re-resolves the whole block and commits derived state. 


    
## Public Attributes Documentation




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
## Public Functions Documentation




### function UPROPERTY [1/9]

```C++
virtual FTryllSamplingOverrides::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|SamplingOverrides",
    meta=(EditCondition="bOverrideTemperature", ClampMin="0.0", ClampMax="2.0", UIMin="0.0", UIMax="2.0")
) = 0
```




<hr>



### function UPROPERTY [2/9]

```C++
virtual FTryllSamplingOverrides::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|SamplingOverrides",
    meta=(EditCondition="bOverrideTopP", ClampMin="0.0", ClampMax="1.0", UIMin="0.0", UIMax="1.0")
) = 0
```




<hr>



### function UPROPERTY [3/9]

```C++
virtual FTryllSamplingOverrides::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|SamplingOverrides",
    meta=(EditCondition="bOverrideTopK", ClampMin="0.0", UIMin="0.0")
) = 0
```




<hr>



### function UPROPERTY [4/9]

```C++
virtual FTryllSamplingOverrides::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|SamplingOverrides",
    meta=(EditCondition="bOverrideMinP", ClampMin="0.0", ClampMax="1.0", UIMin="0.0", UIMax="1.0")
) = 0
```




<hr>



### function UPROPERTY [5/9]

```C++
virtual FTryllSamplingOverrides::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|SamplingOverrides",
    meta=(EditCondition="bOverrideMaxTokens", ClampMin="1.0", UIMin="1.0")
) = 0
```




<hr>



### function UPROPERTY [6/9]

```C++
virtual FTryllSamplingOverrides::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|SamplingOverrides",
    meta=(EditCondition="bOverrideSeed")
) = 0
```




<hr>



### function UPROPERTY [7/9]

```C++
virtual FTryllSamplingOverrides::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|SamplingOverrides",
    meta=(EditCondition="bOverrideRepeatPenalty", ClampMin="0.0", UIMin="0.0")
) = 0
```




<hr>



### function UPROPERTY [8/9]

```C++
virtual FTryllSamplingOverrides::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|SamplingOverrides",
    meta=(EditCondition="bOverridePresencePenalty")
) = 0
```




<hr>



### function UPROPERTY [9/9]

```C++
virtual FTryllSamplingOverrides::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|SamplingOverrides",
    meta=(EditCondition="bOverrideFrequencyPenalty")
) = 0
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/Generated/Nodes/TryllSamplingOverrides.h`

