

# Struct FTryllTextResolve



[**ClassList**](annotated.md) **>** [**FTryllTextResolve**](struct_f_tryll_text_resolve.md)



[More...](#detailed-description)

* `#include <TryllTextResolve.h>`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTextIdRef**](struct_f_tryll_text_resolve_1_1_f_text_id_ref.md) <br> |
























## Public Static Functions

| Type | Name |
| ---: | :--- |
|  TArray&lt; [**FTextIdRef**](struct_f_tryll_text_resolve_1_1_f_text_id_ref.md) &gt; | [**CollectTextIds**](#function-collecttextids) (const [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) & Graph) <br> |
|  bool | [**HasTickedFields**](#function-hastickedfields-12) (const [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) & Graph) <br> |
|  bool | [**HasTickedFields**](#function-hastickedfields-22) (const [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* Params, bool bLiveOnly=false) <br> |
|  bool | [**PrepareForCreate**](#function-prepareforcreate) (const [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) & Graph, UObject \* Outer, [**UTryllTextProvider**](class_u_tryll_text_provider.md) \* Provider, [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) & OutGraph, TArray&lt; [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* &gt; & OutOwned, [**FTryllError**](struct_f_tryll_error.md) & OutError) <br> |
|  bool | [**ResolveParams**](#function-resolveparams) ([**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* Params, [**UTryllTextProvider**](class_u_tryll_text_provider.md) \* Provider, FName Table, const FString & AssetName, const FString & NodeName, bool bLiveOnly, bool & bOutChanged, [**FTryllError**](struct_f_tryll_error.md) & OutError) <br> |


























## Detailed Description


Fills ticked Text IDs before CreateAgent and ChangeParams, and again for non-structural fields when the provider's language changes. Always works on a copy; the authored asset is not modified. 


    
## Public Static Functions Documentation




### function CollectTextIds 

```C++
static TArray< FTextIdRef > FTryllTextResolve::CollectTextIds (
    const FTryllGraphDescription & Graph
) 
```




<hr>



### function HasTickedFields [1/2]

```C++
static bool FTryllTextResolve::HasTickedFields (
    const FTryllGraphDescription & Graph
) 
```




<hr>



### function HasTickedFields [2/2]

```C++
static bool FTryllTextResolve::HasTickedFields (
    const UTryllNodeParamsBase * Params,
    bool bLiveOnly=false
) 
```



True when Params has a ticked field. bLiveOnly ignores fields inside structural containers. 


        

<hr>



### function PrepareForCreate 

```C++
static bool FTryllTextResolve::PrepareForCreate (
    const FTryllGraphDescription & Graph,
    UObject * Outer,
    UTryllTextProvider * Provider,
    FTryllGraphDescription & OutGraph,
    TArray< UTryllNodeParamsBase * > & OutOwned,
    FTryllError & OutError
) 
```



When the graph has a ticked field, duplicate its params onto Outer and replace those strings with provider text. OutOwned is every duplicate, including when resolution fails, so the caller can root or destroy them. A graph with no ticked field is copied by value and OutOwned stays empty. 


        

<hr>



### function ResolveParams 

```C++
static bool FTryllTextResolve::ResolveParams (
    UTryllNodeParamsBase * Params,
    UTryllTextProvider * Provider,
    FName Table,
    const FString & AssetName,
    const FString & NodeName,
    bool bLiveOnly,
    bool & bOutChanged,
    FTryllError & OutError
) 
```



Resolve the ticked fields of Params in place (pass a copy). bLiveOnly skips fields inside structural containers, which cannot change on a live agent. bOutChanged is true when any string that is sent changed. Errors name asset, node, field, table and ID (InvalidParamValue). 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllTextResolve.h`

