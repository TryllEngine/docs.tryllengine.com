

# Struct FTryllModelInfo



[**ClassList**](annotated.md) **>** [**FTryllModelInfo**](struct_f_tryll_model_info.md)



[More...](#detailed-description)

* `#include <TryllModelInfo.h>`





































## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**UPROPERTY**](#function-uproperty-14) (BlueprintReadOnly, Category="Tryll\|Models", meta=(ToolTip="Catalog name of the model as registered in models.json.")) <br> |
|   | [**UPROPERTY**](#function-uproperty-24) (BlueprintReadOnly, Category="Tryll\|Models", meta=(ToolTip="Current status of the model on the server.")) <br> |
|   | [**UPROPERTY**](#function-uproperty-34) (BlueprintReadOnly, Category="Tryll\|Models", meta=(ToolTip="HuggingFace repository id the model can be downloaded from; empty for local-only catalog entries.")) <br> |
| virtual  | [**UPROPERTY**](#function-uproperty-44) (BlueprintReadOnly, Category="Tryll\|Models", meta=(ToolTip="On-disk size in bytes of the downloaded model; zero when the model is not on disk.")) = 0<br> |




























## Detailed Description


Summary information for one model returned by ListModels. 


    
## Public Functions Documentation




### function UPROPERTY [1/4]

```C++
FTryllModelInfo::UPROPERTY (
    BlueprintReadOnly,
    Category="Tryll|Models",
    meta=(ToolTip="Catalog name of the model as registered in models.json.")
) 
```



Catalog name of the model (matches the name in models.json). 


        

<hr>



### function UPROPERTY [2/4]

```C++
FTryllModelInfo::UPROPERTY (
    BlueprintReadOnly,
    Category="Tryll|Models",
    meta=(ToolTip="Current status of the model on the server.")
) 
```



Current status on the server (absent / local / downloading / downloaded / loaded). 


        

<hr>



### function UPROPERTY [3/4]

```C++
FTryllModelInfo::UPROPERTY (
    BlueprintReadOnly,
    Category="Tryll|Models",
    meta=(ToolTip="HuggingFace repository id the model can be downloaded from; empty for local-only catalog entries.")
) 
```



HuggingFace repo id (e.g. "Qwen/Qwen2.5-0.5B-Instruct-GGUF") or empty for local-only models. 


        

<hr>



### function UPROPERTY [4/4]

```C++
virtual FTryllModelInfo::UPROPERTY (
    BlueprintReadOnly,
    Category="Tryll|Models",
    meta=(ToolTip="On-disk size in bytes of the downloaded model; zero when the model is not on disk.")
) = 0
```



On-disk size in bytes when the model has been downloaded. Zero if Status == Absent. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/TryllModelInfo.h`

