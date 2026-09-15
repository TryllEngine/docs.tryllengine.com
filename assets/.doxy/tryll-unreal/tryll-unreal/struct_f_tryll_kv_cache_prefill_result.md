

# Struct FTryllKvCachePrefillResult



[**ClassList**](annotated.md) **>** [**FTryllKvCachePrefillResult**](struct_f_tryll_kv_cache_prefill_result.md)



[More...](#detailed-description)

* `#include <TryllKvCache.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  int64 | [**AgentId**](#variable-agentid)   = `0`<br> |
|  int32 | [**ContextsCreated**](#variable-contextscreated)   = `0`<br> |
|  FString | [**DebugInfo**](#variable-debuginfo)  <br> |
|  int32 | [**EligibleNodeCount**](#variable-eligiblenodecount)   = `0`<br> |
|  ETryllKvCachePrefixStatus | [**PrefixStatus**](#variable-prefixstatus)   = `ETryllKvCachePrefixStatus::NotCurrent`<br> |
|  ETryllKvCacheResidency | [**Residency**](#variable-residency)   = `ETryllKvCacheResidency::Evicted`<br> |
|  int32 | [**TokensDecoded**](#variable-tokensdecoded)   = `0`<br> |
|  int32 | [**TokensReused**](#variable-tokensreused)   = `0`<br> |
|  bool | [**bApplicable**](#variable-bapplicable)   = `false`<br> |












































## Detailed Description


Aggregate result of PrefillKvCache. 


    
## Public Attributes Documentation




### variable AgentId 

```C++
int64 FTryllKvCachePrefillResult::AgentId;
```




<hr>



### variable ContextsCreated 

```C++
int32 FTryllKvCachePrefillResult::ContextsCreated;
```




<hr>



### variable DebugInfo 

```C++
FString FTryllKvCachePrefillResult::DebugInfo;
```




<hr>



### variable EligibleNodeCount 

```C++
int32 FTryllKvCachePrefillResult::EligibleNodeCount;
```




<hr>



### variable PrefixStatus 

```C++
ETryllKvCachePrefixStatus FTryllKvCachePrefillResult::PrefixStatus;
```




<hr>



### variable Residency 

```C++
ETryllKvCacheResidency FTryllKvCachePrefillResult::Residency;
```




<hr>



### variable TokensDecoded 

```C++
int32 FTryllKvCachePrefillResult::TokensDecoded;
```




<hr>



### variable TokensReused 

```C++
int32 FTryllKvCachePrefillResult::TokensReused;
```




<hr>



### variable bApplicable 

```C++
bool FTryllKvCachePrefillResult::bApplicable;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllKvCache.h`

