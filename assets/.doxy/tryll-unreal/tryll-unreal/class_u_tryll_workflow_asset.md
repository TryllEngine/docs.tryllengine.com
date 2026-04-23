

# Class UTryllWorkflowAsset



[**ClassList**](annotated.md) **>** [**UTryllWorkflowAsset**](class_u_tryll_workflow_asset.md)



[More...](#detailed-description)

* `#include <TryllWorkflowAsset.h>`



Inherits the following classes: UPrimaryDataAsset


































## Public Functions

| Type | Name |
| ---: | :--- |
| virtual FPrimaryAssetId | [**GetPrimaryAssetId**](#function-getprimaryassetid) () override const<br> |
|   | [**UPROPERTY**](#function-uproperty) (EditAnywhere, BlueprintReadOnly, Category="Tryll\|Workflow", meta=(ToolTip="Workflow graph stored by this asset. Edit nodes, routes, start node and default model here.")) <br> |




























## Detailed Description


Data asset that stores a Tryll workflow graph description. Equivalent to a Unity ScriptableObject holding node and wire lists.


Usage:
* In the Content Browser: right-click → Miscellaneous → Data Asset → TryllWorkflowAsset.
* Edit nodes and routes in the Details panel.
* Assign this asset to a UTryllAgentComponent's WorkflowAsset property.




Programmatic alternative: populate [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) directly in C++ using [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md), or assign it to UTryllAgentComponent::InlineGraphDescription. 


    
## Public Functions Documentation




### function GetPrimaryAssetId 

```C++
inline virtual FPrimaryAssetId UTryllWorkflowAsset::GetPrimaryAssetId () override const
```




<hr>



### function UPROPERTY 

```C++
UTryllWorkflowAsset::UPROPERTY (
    EditAnywhere,
    BlueprintReadOnly,
    Category="Tryll|Workflow",
    meta=(ToolTip="Workflow graph stored by this asset. Edit nodes, routes, start node and default model here.")
) 
```



Workflow graph stored by this asset. Assign this asset to UTryllAgentComponent::WorkflowAsset to use it. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/tryll-mono/server/client-unreal/Source/TryllClient/Public/TryllWorkflowAsset.h`

