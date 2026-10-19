

# Struct FTryllClassifyIntentEntry



[**ClassList**](annotated.md) **>** [**FTryllClassifyIntentEntry**](struct_f_tryll_classify_intent_entry.md)



[More...](#detailed-description)

* `#include <TryllClassifyIntentEntry.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Id**](#variable-id)  <br> |
|  FString | [**Prompt**](#variable-prompt)  <br> |
|  FString | [**PromptTextId**](#variable-prompttextid)  <br> |
|  bool | [**bPromptLocalized**](#variable-bpromptlocalized)   = `false`<br> |












































## Detailed Description


One intent label for ClassifyIntentLLM. Order in the parent `intents` vector is letter order: entry 0 is A, entry 1 is B, … 


    
## Public Attributes Documentation




### variable Id 

```C++
FString FTryllClassifyIntentEntry::Id;
```



Routing label attached on a win. Never shown to the model. Unique within the parent `intents` vector. 


        

<hr>



### variable Prompt 

```C++
FString FTryllClassifyIntentEntry::Prompt;
```




<hr>



### variable PromptTextId 

```C++
FString FTryllClassifyIntentEntry::PromptTextId;
```



Text ID in the workflow text table. The provider's text is what is sent. 


        

<hr>



### variable bPromptLocalized 

```C++
bool FTryllClassifyIntentEntry::bPromptLocalized;
```



Description the model reads, rendered as "A. &lt;prompt&gt;", "B. &lt;prompt&gt;", … Localizable, but the parent `intents` array is structural, so a ticked id is resolved only when the agent is created. Ticked: PromptTextId is resolved by the text provider and that text is sent. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllClassifyIntentEntry.h`

