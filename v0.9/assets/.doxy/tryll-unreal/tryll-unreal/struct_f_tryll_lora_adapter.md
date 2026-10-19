

# Struct FTryllLoraAdapter



[**ClassList**](annotated.md) **>** [**FTryllLoraAdapter**](struct_f_tryll_lora_adapter.md)



[More...](#detailed-description)

* `#include <TryllLoraAdapter.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  [**FTryllStoragePath**](struct_f_tryll_storage_path.md) | [**Path**](#variable-path)  <br> |
|  float | [**Scale**](#variable-scale)   = `1.0f`<br> |












































## Detailed Description


One LoRA adapter applied to an LLM node's context. The GGUF adapter must have been trained for the node's base model; a file made for another base fails agent creation. Loaded once per model and shared by every context that names the same file. 


    
## Public Attributes Documentation




### variable Path 

```C++
FTryllStoragePath FTryllLoraAdapter::Path;
```



Adapter GGUF, relative to the storage root (like voice samples). 


        

<hr>



### variable Scale 

```C++
float FTryllLoraAdapter::Scale;
```



Adapter strength. 1 = as trained, 0 = off (the adapter is skipped). 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllLoraAdapter.h`

