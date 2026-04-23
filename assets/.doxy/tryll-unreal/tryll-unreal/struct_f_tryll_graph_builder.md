

# Struct FTryllGraphBuilder



[**ClassList**](annotated.md) **>** [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md)



[More...](#detailed-description)

* `#include <TryllGraphDescription.h>`





































## Public Functions

| Type | Name |
| ---: | :--- |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddNode**](#function-addnode) (FString Name, ETryllNodeType Type, TMap&lt; FString, FString &gt; Params={}) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddToolCallNode**](#function-addtoolcallnode) (FString Name, TArray&lt; [**FTryllToolDefinition**](struct_f_tryll_tool_definition.md) &gt; Tools, TMap&lt; FString, FString &gt; Params={}) <br> |
|  [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) | [**Build**](#function-build) () const<br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**SetDefaultModelName**](#function-setdefaultmodelname) (FString Name) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**SetKnowledgePresentation**](#function-setknowledgepresentation) ([**FTryllKnowledgePresentationConfig**](struct_f_tryll_knowledge_presentation_config.md) Config) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**SetStartNode**](#function-setstartnode) (FString Name) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**Wire**](#function-wire) (FString SourceNode, FString ExitName, FString TargetNode) <br> |




























## Detailed Description


Fluent C++ builder for [**FTryllGraphDescription**](struct_f_tryll_graph_description.md). Mirrors Tryll::Client::GraphDescription from client-cpp. Not a USTRUCT — for programmatic C++ use only.


Example: [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) Graph = [**FTryllGraphBuilder()**](struct_f_tryll_graph_builder.md) .AddNode(TEXT("gen"), ETryllNodeType::Generate, {{TEXT("model\_name"), TEXT("mymodel")}}) .Wire(TEXT("gen"), TEXT("done"), TEXT("END")) .SetStartNode(TEXT("gen")) .SetDefaultModelName(TEXT("mymodel")) .Build(); 


    
## Public Functions Documentation




### function AddNode 

```C++
FTryllGraphBuilder & FTryllGraphBuilder::AddNode (
    FString Name,
    ETryllNodeType Type,
    TMap< FString, FString > Params={}
) 
```




<hr>



### function AddToolCallNode 

```C++
FTryllGraphBuilder & FTryllGraphBuilder::AddToolCallNode (
    FString Name,
    TArray< FTryllToolDefinition > Tools,
    TMap< FString, FString > Params={}
) 
```



Convenience method to add a ToolCall node with structured tool definitions. 


        

<hr>



### function Build 

```C++
inline FTryllGraphDescription FTryllGraphBuilder::Build () const
```




<hr>



### function SetDefaultModelName 

```C++
FTryllGraphBuilder & FTryllGraphBuilder::SetDefaultModelName (
    FString Name
) 
```




<hr>



### function SetKnowledgePresentation 

```C++
FTryllGraphBuilder & FTryllGraphBuilder::SetKnowledgePresentation (
    FTryllKnowledgePresentationConfig Config
) 
```



Set the knowledge presentation config. Required when the graph has a Retrieve node. 


        

<hr>



### function SetStartNode 

```C++
FTryllGraphBuilder & FTryllGraphBuilder::SetStartNode (
    FString Name
) 
```




<hr>



### function Wire 

```C++
FTryllGraphBuilder & FTryllGraphBuilder::Wire (
    FString SourceNode,
    FString ExitName,
    FString TargetNode
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/tryll-mono/server/client-unreal/Source/TryllClient/Public/TryllGraphDescription.h`

