

# Struct FTryllGraphDescription



[**ClassList**](annotated.md) **>** [**FTryllGraphDescription**](struct_f_tryll_graph_description.md)



[More...](#detailed-description)

* `#include <TryllGraphDescription.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**DefaultModelName**](#variable-defaultmodelname)  <br> |
|  TArray&lt; [**FTryllNodeDescription**](struct_f_tryll_node_description.md) &gt; | [**Nodes**](#variable-nodes)  <br> |
|  TArray&lt; [**FTryllExitRoute**](struct_f_tryll_exit_route.md) &gt; | [**Routes**](#variable-routes)  <br> |
|  FString | [**StartNode**](#variable-startnode)  <br> |












































## Detailed Description


Complete graph description: nodes, routing, and entry point. Sent inside CreateAgentRequest. 


    
## Public Attributes Documentation




### variable DefaultModelName 

```C++
FString FTryllGraphDescription::DefaultModelName;
```



Fallback model name for nodes that do not specify their own "model\_name" param. Empty means every Generate node must supply its own. 


        

<hr>



### variable Nodes 

```C++
TArray<FTryllNodeDescription> FTryllGraphDescription::Nodes;
```




<hr>



### variable Routes 

```C++
TArray<FTryllExitRoute> FTryllGraphDescription::Routes;
```




<hr>



### variable StartNode 

```C++
FString FTryllGraphDescription::StartNode;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo/server/client-unreal/Source/TryllClient/Public/TryllGraphDescription.h`

