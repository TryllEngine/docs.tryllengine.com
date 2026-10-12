

# Struct Tryll::Client::AgentKvCachePrefillResult



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**AgentKvCachePrefillResult**](struct_tryll_1_1_client_1_1_agent_kv_cache_prefill_result.md)








Inherits the following classes: [Tryll::Client::AgentKvCacheStatus](struct_tryll_1_1_client_1_1_agent_kv_cache_status.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::uint32\_t | [**contextsCreated**](#variable-contextscreated)   = `0`<br> |
|  std::uint32\_t | [**tokensDecoded**](#variable-tokensdecoded)   = `0`<br> |
|  std::uint32\_t | [**tokensReused**](#variable-tokensreused)   = `0`<br> |


## Public Attributes inherited from Tryll::Client::AgentKvCacheStatus

See [Tryll::Client::AgentKvCacheStatus](struct_tryll_1_1_client_1_1_agent_kv_cache_status.md)

| Type | Name |
| ---: | :--- |
|  std::uint64\_t | [**agentId**](struct_tryll_1_1_client_1_1_agent_kv_cache_status.md#variable-agentid)   = `0`<br> |
|  bool | [**applicable**](struct_tryll_1_1_client_1_1_agent_kv_cache_status.md#variable-applicable)   = `false`<br> |
|  std::string | [**debugInfo**](struct_tryll_1_1_client_1_1_agent_kv_cache_status.md#variable-debuginfo)  <br> |
|  std::uint32\_t | [**eligibleNodeCount**](struct_tryll_1_1_client_1_1_agent_kv_cache_status.md#variable-eligiblenodecount)   = `0`<br> |
|  AgentKvCachePrefixStatus | [**prefixStatus**](struct_tryll_1_1_client_1_1_agent_kv_cache_status.md#variable-prefixstatus)   = `AgentKvCachePrefixStatus::NotCurrent`<br> |
|  AgentKvCacheResidency | [**residency**](struct_tryll_1_1_client_1_1_agent_kv_cache_status.md#variable-residency)   = `AgentKvCacheResidency::Evicted`<br> |






















































































## Public Attributes Documentation




### variable contextsCreated 

```C++
std::uint32_t Tryll::Client::AgentKvCachePrefillResult::contextsCreated;
```




<hr>



### variable tokensDecoded 

```C++
std::uint32_t Tryll::Client::AgentKvCachePrefillResult::tokensDecoded;
```




<hr>



### variable tokensReused 

```C++
std::uint32_t Tryll::Client::AgentKvCachePrefillResult::tokensReused;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/cpp/include/tryll/AgentKvCache.h`

