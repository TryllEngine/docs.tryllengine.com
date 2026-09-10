

# Struct FTryllContextOverrides



[**ClassList**](annotated.md) **>** [**FTryllContextOverrides**](struct_f_tryll_context_overrides.md)



[More...](#detailed-description)

* `#include <TryllContextOverrides.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  ETryllKvCacheType | [**KvCacheType**](#variable-kvcachetype)   = `ETryllKvCacheType::Inherit`<br> |
|  int32 | [**NBatch**](#variable-nbatch)   = `0`<br> |
|  int32 | [**NOutputsMax**](#variable-noutputsmax)   = `0`<br> |
|  int32 | [**NUbatch**](#variable-nubatch)   = `0`<br> |
|  bool | [**OffloadKqv**](#variable-offloadkqv)   = `0`<br> |
|  bool | [**bOverrideNBatch**](#variable-boverridenbatch)   = `false`<br> |
|  bool | [**bOverrideNOutputsMax**](#variable-boverridenoutputsmax)   = `false`<br> |
|  bool | [**bOverrideNUbatch**](#variable-boverridenubatch)   = `false`<br> |
|  bool | [**bOverrideOffloadKqv**](#variable-boverrideoffloadkqv)   = `false`<br> |












































## Detailed Description


Backend context construction. Null fields inherit ModelVariant then server config, except `n_outputs_max` which CreateContext defaults to 1. Structural at the field: these are baked into llama\_context\_params at context creation, so a change means a new context and a lost KV cache. 


    
## Public Attributes Documentation




### variable KvCacheType 

```C++
ETryllKvCacheType FTryllContextOverrides::KvCacheType;
```



KV cache dtype. Inherit = ModelVariant, then q8\_0. 


        

<hr>



### variable NBatch 

```C++
int32 FTryllContextOverrides::NBatch;
```




<hr>



### variable NOutputsMax 

```C++
int32 FTryllContextOverrides::NOutputsMax;
```




<hr>



### variable NUbatch 

```C++
int32 FTryllContextOverrides::NUbatch;
```




<hr>



### variable OffloadKqv 

```C++
bool FTryllContextOverrides::OffloadKqv;
```




<hr>



### variable bOverrideNBatch 

```C++
bool FTryllContextOverrides::bOverrideNBatch;
```



Logical llama\_decode submission ceiling. 


        

<hr>



### variable bOverrideNOutputsMax 

```C++
bool FTryllContextOverrides::bOverrideNOutputsMax;
```



Logits buffer positions. Engine default is 1 for every current LLM node; this field is an escape hatch if a future path needs more. 


        

<hr>



### variable bOverrideNUbatch 

```C++
bool FTryllContextOverrides::bOverrideNUbatch;
```



Physical micro-batch size that sizes activation buffers. 


        

<hr>



### variable bOverrideOffloadKqv 

```C++
bool FTryllContextOverrides::bOverrideOffloadKqv;
```



Place the KV cache on GPU. Veto only — a node can force false but cannot grant offload to a RAM-placed model. Null = inherit server/placement. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllContextOverrides.h`

