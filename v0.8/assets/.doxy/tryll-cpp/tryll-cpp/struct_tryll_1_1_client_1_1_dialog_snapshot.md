

# Struct Tryll::Client::DialogSnapshot



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**DialogSnapshot**](struct_tryll_1_1_client_1_1_dialog_snapshot.md)


























## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::uint32\_t | [**formatVersion**](#variable-formatversion)   = `1`<br> |
|  std::string | [**graphSignature**](#variable-graphsignature)  <br> |
|  std::vector&lt; [**DialogInteractionSnapshot**](struct_tryll_1_1_client_1_1_dialog_interaction_snapshot.md) &gt; | [**interactions**](#variable-interactions)  <br> |
|  std::uint64\_t | [**nextExchangeId**](#variable-nextexchangeid)   = `1`<br> |
|  std::uint64\_t | [**nextToolCallSeq**](#variable-nexttoolcallseq)   = `0`<br> |
|  std::vector&lt; [**DialogProjectionWindow**](struct_tryll_1_1_client_1_1_dialog_projection_window.md) &gt; | [**projectionWindows**](#variable-projectionwindows)  <br> |
|  std::string | [**saveId**](#variable-saveid)  <br> |
|  std::uint64\_t | [**savedAtMs**](#variable-savedatms)   = `0`<br> |












































## Public Attributes Documentation




### variable formatVersion 

```C++
std::uint32_t Tryll::Client::DialogSnapshot::formatVersion;
```




<hr>



### variable graphSignature 

```C++
std::string Tryll::Client::DialogSnapshot::graphSignature;
```




<hr>



### variable interactions 

```C++
std::vector<DialogInteractionSnapshot> Tryll::Client::DialogSnapshot::interactions;
```




<hr>



### variable nextExchangeId 

```C++
std::uint64_t Tryll::Client::DialogSnapshot::nextExchangeId;
```




<hr>



### variable nextToolCallSeq 

```C++
std::uint64_t Tryll::Client::DialogSnapshot::nextToolCallSeq;
```




<hr>



### variable projectionWindows 

```C++
std::vector<DialogProjectionWindow> Tryll::Client::DialogSnapshot::projectionWindows;
```




<hr>



### variable saveId 

```C++
std::string Tryll::Client::DialogSnapshot::saveId;
```




<hr>



### variable savedAtMs 

```C++
std::uint64_t Tryll::Client::DialogSnapshot::savedAtMs;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/cpp/include/tryll/DialogSnapshot.h`

