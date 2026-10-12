

# Struct FTryllClassifyIntentEntry



[**ClassList**](annotated.md) **>** [**FTryllClassifyIntentEntry**](struct_f_tryll_classify_intent_entry.md)



[More...](#detailed-description)

* `#include <TryllClassifyIntentEntry.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Id**](#variable-id)  <br> |
|  FString | [**Prompt**](#variable-prompt)  <br> |
|  bool | [**bOverrideId**](#variable-boverrideid)   = `false`<br> |
|  bool | [**bOverridePrompt**](#variable-boverrideprompt)   = `false`<br> |












































## Detailed Description


One intent label for ClassifyIntentLLM. Order in the parent `intents` vector is letter order: entry 0 is A, entry 1 is B, … 


    
## Public Attributes Documentation




### variable Id 

```C++
FString FTryllClassifyIntentEntry::Id;
```




<hr>



### variable Prompt 

```C++
FString FTryllClassifyIntentEntry::Prompt;
```




<hr>



### variable bOverrideId 

```C++
bool FTryllClassifyIntentEntry::bOverrideId;
```



Routing label attached on a win. Never shown to the model. Unique within the parent `intents` vector. 


        

<hr>



### variable bOverridePrompt 

```C++
bool FTryllClassifyIntentEntry::bOverridePrompt;
```



Description the model reads, rendered as "A. &lt;prompt&gt;", "B. &lt;prompt&gt;", … 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllClassifyIntentEntry.h`

