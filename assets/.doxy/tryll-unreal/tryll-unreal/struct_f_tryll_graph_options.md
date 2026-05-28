

# Struct FTryllGraphOptions



[**ClassList**](annotated.md) **>** [**FTryllGraphOptions**](struct_f_tryll_graph_options.md)



[More...](#detailed-description)

* `#include <TryllGraphDescription.h>`







































## Public Static Functions

| Type | Name |
| ---: | :--- |
|  TArray&lt; FString &gt; | [**GetNodeNameOptions**](#function-getnodenameoptions) (const [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) & Graph) <br> |


























## Detailed Description


Shared helpers for Details-panel dropdowns that enumerate node names. Used by GetExitTargetOptions (on [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md)) and GetStartNodeOptions (on [**UTryllWorkflowAsset**](class_u_tryll_workflow_asset.md) / UTryllAgentComponent). 


    
## Public Static Functions Documentation




### function GetNodeNameOptions 

```C++
static TArray< FString > FTryllGraphOptions::GetNodeNameOptions (
    const FTryllGraphDescription & Graph
) 
```



Returns all non-empty node names from Graph.Nodes, in authoring order. No END sentinel. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/TryllGraphDescription.h`

