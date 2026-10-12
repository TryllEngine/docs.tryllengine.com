

# Struct FTryllKvCacheEvictResult



[**ClassList**](annotated.md) **>** [**FTryllKvCacheEvictResult**](struct_f_tryll_kv_cache_evict_result.md)



[More...](#detailed-description)

* `#include <TryllKvCache.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  int64 | [**AgentId**](#variable-agentid)   = `0`<br> |
|  int32 | [**ContextsEvicted**](#variable-contextsevicted)   = `0`<br> |
|  FString | [**DebugInfo**](#variable-debuginfo)  <br> |
|  int32 | [**EligibleNodeCount**](#variable-eligiblenodecount)   = `0`<br> |
|  ETryllKvCachePrefixStatus | [**PrefixStatus**](#variable-prefixstatus)   = `ETryllKvCachePrefixStatus::NotCurrent`<br> |
|  ETryllKvCacheResidency | [**Residency**](#variable-residency)   = `ETryllKvCacheResidency::Evicted`<br> |
|  bool | [**bApplicable**](#variable-bapplicable)   = `false`<br> |












































## Detailed Description


Aggregate result of EvictKvCache. 


    
## Public Attributes Documentation




### variable AgentId 

```C++
int64 FTryllKvCacheEvictResult::AgentId;
```




<hr>



### variable ContextsEvicted 

```C++
int32 FTryllKvCacheEvictResult::ContextsEvicted;
```




<hr>



### variable DebugInfo 

```C++
FString FTryllKvCacheEvictResult::DebugInfo;
```




<hr>



### variable EligibleNodeCount 

```C++
int32 FTryllKvCacheEvictResult::EligibleNodeCount;
```




<hr>



### variable PrefixStatus 

```C++
ETryllKvCachePrefixStatus FTryllKvCacheEvictResult::PrefixStatus;
```




<hr>



### variable Residency 

```C++
ETryllKvCacheResidency FTryllKvCacheEvictResult::Residency;
```




<hr>



### variable bApplicable 

```C++
bool FTryllKvCacheEvictResult::bApplicable;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllKvCache.h`

