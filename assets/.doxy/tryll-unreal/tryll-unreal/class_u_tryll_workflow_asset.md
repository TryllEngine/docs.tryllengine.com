

# Class UTryllWorkflowAsset



[**ClassList**](annotated.md) **>** [**UTryllWorkflowAsset**](class_u_tryll_workflow_asset.md)



[More...](#detailed-description)

* `#include <TryllWorkflowAsset.h>`



Inherits the following classes: UPrimaryDataAsset


















## Public Attributes

| Type | Name |
| ---: | :--- |
|  [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) | [**Graph**](#variable-graph)  <br> |
|  TArray&lt; [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) &gt; | [**Variables**](#variable-variables)  <br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
| virtual FPrimaryAssetId | [**GetPrimaryAssetId**](#function-getprimaryassetid) () override const<br> |
|  TArray&lt; FString &gt; | [**GetStartNodeOptions**](#function-getstartnodeoptions) () const<br> |




























## Detailed Description


Data asset that stores a Tryll workflow graph description. Equivalent to a Unity ScriptableObject holding node and wire lists.


Usage:
* In the Content Browser: right-click → Tryll → Workflow Asset.
* Edit nodes and routes in the Details panel.
* Assign this asset to a [**UTryllAgentComponent**](class_u_tryll_agent_component.md)'s WorkflowAsset property.




Programmatic alternative: populate [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) directly in C++ using [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) and pass it to [**UTryllSubsystem::RequestCreateAgent**](class_u_tryll_subsystem.md#function-requestcreateagent). ([**UTryllAgentComponent::InlineGraphDescription**](class_u_tryll_agent_component.md#variable-inlinegraphdescription) also accepts a graph, but it is temporary and will be removed in a future release.)


Editor / cook: IsDataValid + PreSave(IsCooking) fail when the graph references a model that is not marked Production in the Tryll Model Manifest (so packaging cannot silently omit it). 


    
## Public Attributes Documentation




### variable Graph 

```C++
FTryllGraphDescription UTryllWorkflowAsset::Graph;
```



Workflow graph stored by this asset. Assign this asset to [**UTryllAgentComponent::WorkflowAsset**](class_u_tryll_agent_component.md#variable-workflowasset) to use it. 


        

<hr>



### variable Variables 

```C++
TArray<FTryllVariableDecl> UTryllWorkflowAsset::Variables;
```



Declared per-agent Variables for agents created from this asset — the complete declaration (name, type, initial value) sent once in CreateAgentRequest. Consumable from Mustache templates as {{var.&lt;name&gt;}} and the Retrieve/ ClassifyIntent filter grammar as {"var": "&lt;name&gt;"}. A [**UTryllAgentComponent**](class_u_tryll_agent_component.md) may override initial values (not name/type) via its own VariableOverrides (Material-Instance pattern). 


        

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
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllWorkflowAsset.h`

