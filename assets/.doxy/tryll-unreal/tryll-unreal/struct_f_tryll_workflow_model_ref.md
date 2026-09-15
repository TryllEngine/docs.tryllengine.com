

# Struct FTryllWorkflowModelRef



[**ClassList**](annotated.md) **>** [**FTryllWorkflowModelRef**](struct_f_tryll_workflow_model_ref.md)



[More...](#detailed-description)

* `#include <TryllWorkflowModelRefs.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**FieldName**](#variable-fieldname)  <br> |
|  FString | [**ModelName**](#variable-modelname)  <br> |
|  FString | [**ModelType**](#variable-modeltype)  <br> |
|  FString | [**NodeName**](#variable-nodename)  <br> |












































## Detailed Description


Collects model-name references from a workflow graph and compares them against the project's TryllModelManifest. Used by UTryllWorkflowAsset::IsDataValid / PreSave (cook) so a packaged build cannot silently omit a referenced model.


Covers workflow assets. 


    
## Public Attributes Documentation




### variable FieldName 

```C++
FString FTryllWorkflowModelRef::FieldName;
```




<hr>



### variable ModelName 

```C++
FString FTryllWorkflowModelRef::ModelName;
```




<hr>



### variable ModelType 

```C++
FString FTryllWorkflowModelRef::ModelType;
```



"language" \| "embedding" \| "stt" \| "tts" \| "vad" 


        

<hr>



### variable NodeName 

```C++
FString FTryllWorkflowModelRef::NodeName;
```



Empty for graph-level DefaultModelName. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllWorkflowModelRefs.h`

