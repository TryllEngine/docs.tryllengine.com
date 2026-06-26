

# Struct FTryllGraphDescription



[**ClassList**](annotated.md) **>** [**FTryllGraphDescription**](struct_f_tryll_graph_description.md)



[More...](#detailed-description)

* `#include <TryllGraphDescription.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  [**FTryllModelName**](struct_f_tryll_model_name.md) | [**DefaultModelName**](#variable-defaultmodelname)  <br> |
|  TArray&lt; [**FTryllNodeDescription**](struct_f_tryll_node_description.md) &gt; | [**Nodes**](#variable-nodes)  <br> |
|  FString | [**StartNode**](#variable-startnode)  <br> |












































## Detailed Description


Complete graph description: nodes and entry point. Sent inside CreateAgentRequest.


Protocol v2: wiring lives on each node's typed [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) in fields marked with `meta=(GetOptions="GetExitTargetOptions")` (one per declared exit). Empty value = END (turn finishes after the node). There is no separate Routes list. 


    
## Public Attributes Documentation




### variable DefaultModelName 

```C++
FTryllModelName FTryllGraphDescription::DefaultModelName;
```



Fallback model name for nodes that do not specify their own "model\_name" param. Empty means every Generate node must supply its own. 


        

<hr>



### variable Nodes 

```C++
TArray<FTryllNodeDescription> FTryllGraphDescription::Nodes;
```




<hr>



### variable StartNode 

```C++
FString FTryllGraphDescription::StartNode;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllGraphDescription.h`

