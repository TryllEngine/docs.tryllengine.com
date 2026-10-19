

# Struct Tryll::Client::GameMemorySnapshot



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**GameMemorySnapshot**](struct_tryll_1_1_client_1_1_game_memory_snapshot.md)


























## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::uint32\_t | [**formatVersion**](#variable-formatversion)   = `kGameMemoryFormatVersion`<br> |
|  std::string | [**saveId**](#variable-saveid)  <br> |
|  std::uint64\_t | [**savedAtMs**](#variable-savedatms)   = `0`<br> |
|  std::vector&lt; [**MutableKbSnapshot**](struct_tryll_1_1_client_1_1_mutable_kb_snapshot.md) &gt; | [**stores**](#variable-stores)  <br> |












































## Public Attributes Documentation




### variable formatVersion 

```C++
std::uint32_t Tryll::Client::GameMemorySnapshot::formatVersion;
```




<hr>



### variable saveId 

```C++
std::string Tryll::Client::GameMemorySnapshot::saveId;
```




<hr>



### variable savedAtMs 

```C++
std::uint64_t Tryll::Client::GameMemorySnapshot::savedAtMs;
```




<hr>



### variable stores 

```C++
std::vector<MutableKbSnapshot> Tryll::Client::GameMemorySnapshot::stores;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/cpp/include/tryll/MkbSnapshot.h`

