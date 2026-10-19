

# File TryllMemorySnapshot.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllMemorySnapshot.h**](_tryll_memory_snapshot_8h.md)





* `#include "CoreMinimal.h"`
* `#include "TryllMemorySnapshot.generated.h"`













## Namespaces

| Type | Name |
| ---: | :--- |
| namespace | [**Tryll**](namespace_tryll.md) <br> |


## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTryllMemoryComponent**](struct_f_tryll_memory_component.md) <br> |
| struct | [**FTryllMemoryComponentDevice**](struct_f_tryll_memory_component_device.md) <br> |
| struct | [**FTryllMemoryDevice**](struct_f_tryll_memory_device.md) <br> |
| struct | [**FTryllMemorySnapshot**](struct_f_tryll_memory_snapshot.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| enum uint8 | [**ETryllInferenceConsumerClass**](#enum-etryllinferenceconsumerclass)  <br> |
| enum uint8 | [**ETryllMemoryOwnerKind**](#enum-etryllmemoryownerkind)  <br> |
| enum uint8 | [**ETryllMemoryTier**](#enum-etryllmemorytier)  <br> |
















































## Public Types Documentation




### enum ETryllInferenceConsumerClass 

```C++
enum ETryllInferenceConsumerClass {
    Unspecified = 0,
    InteractiveVisible = 1,
    InteractiveBuffered = 2,
    Background = 3
};
```



Scheduler class of the last turn on a language-model context. Describes work, not the agent — always read alongside LastTurnAtMs. 


        

<hr>



### enum ETryllMemoryOwnerKind 

```C++
enum ETryllMemoryOwnerKind {
    Process = 0,
    Device = 1,
    Model = 2,
    AgentNode = 3,
    Storage = 4,
    Unattributed = 5
};
```



What owns a component's bytes. Weights belong to a model; KV to an agent × node. 


        

<hr>



### enum ETryllMemoryTier 

```C++
enum ETryllMemoryTier {
    Measured = 0,
    Derived = 1,
    Estimated = 2
};
```



How much to trust a reported memory figure. Rendering a number without its tier is a client bug — an unmarked estimate will be compared against Task Manager and believed. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllMemorySnapshot.h`

