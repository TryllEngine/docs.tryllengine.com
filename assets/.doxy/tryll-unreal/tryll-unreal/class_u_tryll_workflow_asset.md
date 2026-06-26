

# Class UTryllWorkflowAsset



[**ClassList**](annotated.md) **>** [**UTryllWorkflowAsset**](class_u_tryll_workflow_asset.md)



[More...](#detailed-description)

* `#include <TryllWorkflowAsset.h>`



Inherits the following classes: UPrimaryDataAsset


















## Public Attributes

| Type | Name |
| ---: | :--- |
|  [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) | [**Graph**](#variable-graph)  <br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
| virtual FPrimaryAssetId | [**GetPrimaryAssetId**](#function-getprimaryassetid) () override const<br> |
|  TArray&lt; FString &gt; | [**GetStartNodeOptions**](#function-getstartnodeoptions) () const<br> |




























## Detailed Description


Data asset that stores a Tryll workflow graph description. Equivalent to a Unity ScriptableObject holding node and wire lists.


Usage:
* In the Content Browser: right-click → Miscellaneous → Data Asset → TryllWorkflowAsset.
* Edit nodes and routes in the Details panel.
* Assign this asset to a [**UTryllAgentComponent**](class_u_tryll_agent_component.md)'s WorkflowAsset property.




Programmatic alternative: populate [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) directly in C++ using [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md), or assign it to [**UTryllAgentComponent::InlineGraphDescription**](class_u_tryll_agent_component.md#variable-inlinegraphdescription). 


    
## Public Attributes Documentation




### variable Graph 

```C++
FTryllGraphDescription UTryllWorkflowAsset::Graph;
```



Workflow graph stored by this asset. Assign this asset to [**UTryllAgentComponent::WorkflowAsset**](class_u_tryll_agent_component.md#variable-workflowasset) to use it. 


        

<hr>
## Public Functions Documentation




### function GetPrimaryAssetId 

```C++
inline virtual FPrimaryAssetId UTryllWorkflowAsset::GetPrimaryAssetId () override const
```




<hr>



### function GetStartNodeOptions 

```C++
inline TArray< FString > UTryllWorkflowAsset::GetStartNodeOptions () const
```



Options provider for FTryllGraphDescription::StartNode dropdown in the Details panel. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllWorkflowAsset.h`

