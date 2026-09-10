

# Struct Tryll::Client::AgentKvCacheStatus



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**AgentKvCacheStatus**](struct_tryll_1_1_client_1_1_agent_kv_cache_status.md)










Inherited by the following classes: [Tryll::Client::AgentKvCacheEvictResult](struct_tryll_1_1_client_1_1_agent_kv_cache_evict_result.md),  [Tryll::Client::AgentKvCachePrefillResult](struct_tryll_1_1_client_1_1_agent_kv_cache_prefill_result.md)
















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::uint64\_t | [**agentId**](#variable-agentid)   = `0`<br> |
|  bool | [**applicable**](#variable-applicable)   = `false`<br> |
|  std::string | [**debugInfo**](#variable-debuginfo)  <br> |
|  std::uint32\_t | [**eligibleNodeCount**](#variable-eligiblenodecount)   = `0`<br> |
|  AgentKvCachePrefixStatus | [**prefixStatus**](#variable-prefixstatus)   = `AgentKvCachePrefixStatus::NotCurrent`<br> |
|  AgentKvCacheResidency | [**residency**](#variable-residency)   = `AgentKvCacheResidency::Evicted`<br> |












































## Public Attributes Documentation




### variable agentId 

```C++
std::uint64_t Tryll::Client::AgentKvCacheStatus::agentId;
```




<hr>



### variable applicable 

```C++
bool Tryll::Client::AgentKvCacheStatus::applicable;
```




<hr>



### variable debugInfo 

```C++
std::string Tryll::Client::AgentKvCacheStatus::debugInfo;
```




<hr>



### variable eligibleNodeCount 

```C++
std::uint32_t Tryll::Client::AgentKvCacheStatus::eligibleNodeCount;
```




<hr>



### variable prefixStatus 

```C++
AgentKvCachePrefixStatus Tryll::Client::AgentKvCacheStatus::prefixStatus;
```




<hr>



### variable residency 

```C++
AgentKvCacheResidency Tryll::Client::AgentKvCacheStatus::residency;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/AgentKvCache.h`

