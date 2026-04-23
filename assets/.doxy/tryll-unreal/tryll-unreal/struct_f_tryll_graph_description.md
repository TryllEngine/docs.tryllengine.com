

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
|  bool | [**bHasKnowledgePresentation**](#variable-bhasknowledgepresentation)   = `false`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**UPROPERTY**](#function-uproperty) (EditAnywhere, BlueprintReadWrite, Category="Tryll\|Knowledge", meta=(EditCondition="bHasKnowledgePresentation")) <br> |




























## Detailed Description


Complete graph description: nodes, routing, and entry point. Sent inside CreateAgentRequest. Now includes an optional KnowledgePresentationConfig for graphs with Retrieve nodes. 


    
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



### variable bHasKnowledgePresentation 

```C++
bool FTryllGraphDescription::bHasKnowledgePresentation;
```



Knowledge presentation config. Required when the graph contains a Retrieve node; ignored otherwise. bHasKnowledgePresentation must be true to include it. 


        

<hr>
## Public Functions Documentation




### function UPROPERTY 

```C++
FTryllGraphDescription::UPROPERTY (
    EditAnywhere,
    BlueprintReadWrite,
    Category="Tryll|Knowledge",
    meta=(EditCondition="bHasKnowledgePresentation")
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/tryll-mono/server/client-unreal/Source/TryllClient/Public/TryllGraphDescription.h`

