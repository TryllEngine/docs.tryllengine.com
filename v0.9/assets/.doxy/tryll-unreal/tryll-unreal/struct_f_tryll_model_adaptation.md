

# Struct FTryllModelAdaptation



[**ClassList**](annotated.md) **>** [**FTryllModelAdaptation**](struct_f_tryll_model_adaptation.md)



[More...](#detailed-description)

* `#include <TryllModelAdaptation.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  TArray&lt; [**FTryllLoraAdapter**](struct_f_tryll_lora_adapter.md) &gt; | [**Loras**](#variable-loras)  <br> |












































## Detailed Description


Changes to how the model behaves for one LLM node, applied to the node's llama context after it is created. Kept apart from ContextOverrides, which only sizes memory and batching. Control vectors (steering) will join here. 


    
## Public Attributes Documentation




### variable Loras 

```C++
TArray<FTryllLoraAdapter> FTryllModelAdaptation::Loras;
```



LoRA adapters. Their deltas add, so order does not matter. Empty = none. At most 8, and each file at most once. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllModelAdaptation.h`

