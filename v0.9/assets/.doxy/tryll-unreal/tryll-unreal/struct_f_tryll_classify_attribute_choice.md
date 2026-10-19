

# Struct FTryllClassifyAttributeChoice



[**ClassList**](annotated.md) **>** [**FTryllClassifyAttributeChoice**](struct_f_tryll_classify_attribute_choice.md)



[More...](#detailed-description)

* `#include <TryllClassifyAttributeChoice.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Id**](#variable-id)  <br> |
|  FString | [**Prompt**](#variable-prompt)  <br> |
|  bool | [**bTriggers**](#variable-btriggers)   = `false`<br> |












































## Detailed Description


One option of a Choice or Score attribute. Boolean does not use this table: its menu is fixed as A=True, B=False. 


    
## Public Attributes Documentation




### variable Id 

```C++
FString FTryllClassifyAttributeChoice::Id;
```



Routing label. Never shown to the model. Unique within the attribute. 


        

<hr>



### variable Prompt 

```C++
FString FTryllClassifyAttributeChoice::Prompt;
```



Description the model reads, rendered as "A. &lt;prompt&gt;". 


        

<hr>



### variable bTriggers 

```C++
bool FTryllClassifyAttributeChoice::bTriggers;
```



Choice only. When this option is the value written to the slot, the node takes triggered\_exit. Must stay false on a Score ladder. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllClassifyAttributeChoice.h`

