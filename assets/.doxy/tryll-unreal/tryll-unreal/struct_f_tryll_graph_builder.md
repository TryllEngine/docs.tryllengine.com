

# Struct FTryllGraphBuilder



[**ClassList**](annotated.md) **>** [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md)



[More...](#detailed-description)

* `#include <TryllGraphDescription.h>`





































## Public Functions

| Type | Name |
| ---: | :--- |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddNode**](#function-addnode) (FString Name, [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* Params) <br> |
|  [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) | [**Build**](#function-build) () const<br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**SetDefaultModelName**](#function-setdefaultmodelname) (FString Name) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**SetStartNode**](#function-setstartnode) (FString Name) <br> |




























## Detailed Description


Fluent C++ builder for [**FTryllGraphDescription**](struct_f_tryll_graph_description.md). Typed AddXxx methods are declared in TryllGraphBuilder.Nodes.h (generated). Not a USTRUCT — for programmatic C++ use only.


Example: UTryllGenerateParams\* P = UTryllNodeParamsFactory::MakeGenerateParams(this); P-&gt;ModelName = TEXT("qwen2.5-0.5b-instruct"); P-&gt;SystemPrompt = TEXT("You are helpful.");


// Protocol v2: wiring lives on each node's params (e.g. P-&gt;DefaultExit). [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) Graph = [**FTryllGraphBuilder()**](struct_f_tryll_graph_builder.md) .AddGenerate(TEXT("gen"), P) // P-&gt;DefaultExit defaults to "" = END .SetStartNode(TEXT("gen")) .Build(); 


    
## Public Functions Documentation




### function AddNode 

```C++
FTryllGraphBuilder & FTryllGraphBuilder::AddNode (
    FString Name,
    UTryllNodeParamsBase * Params
) 
```




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



### function SetStartNode 

```C++
FTryllGraphBuilder & FTryllGraphBuilder::SetStartNode (
    FString Name
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/TryllGraphDescription.h`

