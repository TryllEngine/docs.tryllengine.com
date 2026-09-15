

# File TryllKvCache.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllKvCache.h**](_tryll_kv_cache_8h.md)





* `#include "CoreMinimal.h"`
* `#include "TryllKvCache.generated.h"`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTryllKvCacheEvictResult**](struct_f_tryll_kv_cache_evict_result.md) <br> |
| struct | [**FTryllKvCachePrefillResult**](struct_f_tryll_kv_cache_prefill_result.md) <br> |
| struct | [**FTryllKvCacheStatus**](struct_f_tryll_kv_cache_status.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| enum uint8 | [**ETryllKvCacheInitialization**](#enum-etryllkvcacheinitialization)  <br> |
| enum uint8 | [**ETryllKvCachePrefixStatus**](#enum-etryllkvcacheprefixstatus)  <br> |
| enum uint8 | [**ETryllKvCacheResidency**](#enum-etryllkvcacheresidency)  <br> |
















































## Public Types Documentation




### enum ETryllKvCacheInitialization 

```C++
enum ETryllKvCacheInitialization {
    AllocateOnly,
    Prefill,
    DeferAllocation
};
```



How language-model KV contexts are materialised during CreateAgent. 


        

<hr>



### enum ETryllKvCachePrefixStatus 

```C++
enum ETryllKvCachePrefixStatus {
    NotCurrent,
    Current
};
```



Whether every eligible language-model node matches the reusable prefix. 


        

<hr>



### enum ETryllKvCacheResidency 

```C++
enum ETryllKvCacheResidency {
    Evicted,
    Resident
};
```



Whether every eligible language-model node has a backend KV context. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllKvCache.h`

