

# Struct FTryllKvCacheStatus



[**ClassList**](annotated.md) **>** [**FTryllKvCacheStatus**](struct_f_tryll_kv_cache_status.md)



[More...](#detailed-description)

* `#include <TryllKvCache.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  int64 | [**AgentId**](#variable-agentid)   = `0`<br> |
|  FString | [**DebugInfo**](#variable-debuginfo)  <br> |
|  int32 | [**EligibleNodeCount**](#variable-eligiblenodecount)   = `0`<br> |
|  ETryllKvCachePrefixStatus | [**PrefixStatus**](#variable-prefixstatus)   = `ETryllKvCachePrefixStatus::NotCurrent`<br> |
|  ETryllKvCacheResidency | [**Residency**](#variable-residency)   = `ETryllKvCacheResidency::Evicted`<br> |
|  bool | [**bApplicable**](#variable-bapplicable)   = `false`<br> |












































## Detailed Description


Aggregate KV-cache lifecycle state for an agent. 


    
## Public Attributes Documentation




### variable AgentId 

```C++
int64 FTryllKvCacheStatus::AgentId;
```




<hr>



### variable DebugInfo 

```C++
FString FTryllKvCacheStatus::DebugInfo;
```




<hr>



### variable EligibleNodeCount 

```C++
int32 FTryllKvCacheStatus::EligibleNodeCount;
```




<hr>



### variable PrefixStatus 

```C++
ETryllKvCachePrefixStatus FTryllKvCacheStatus::PrefixStatus;
```




<hr>



### variable Residency 

```C++
ETryllKvCacheResidency FTryllKvCacheStatus::Residency;
```




<hr>



### variable bApplicable 

```C++
bool FTryllKvCacheStatus::bApplicable;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllKvCache.h`

