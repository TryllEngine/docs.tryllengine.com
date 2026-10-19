

# Struct FTryllClassifyAttributeDef



[**ClassList**](annotated.md) **>** [**FTryllClassifyAttributeDef**](struct_f_tryll_classify_attribute_def.md)



[More...](#detailed-description)

* `#include <TryllClassifyAttributeDef.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  ETryllBooleanTrigger | [**BooleanTrigger**](#variable-booleantrigger)   = `ETryllBooleanTrigger::Never`<br> |
|  TArray&lt; [**FTryllClassifyAttributeChoice**](struct_f_tryll_classify_attribute_choice.md) &gt; | [**Choices**](#variable-choices)  <br> |
|  ETryllAttributeKind | [**Kind**](#variable-kind)   = `ETryllAttributeKind::Choice`<br> |
|  float | [**Margin**](#variable-margin)   = `0.15f`<br> |
|  FString | [**Name**](#variable-name)  <br> |
|  float | [**ScoreThreshold**](#variable-scorethreshold)   = `0.5f`<br> |
|  ETryllScoreTrigger | [**ScoreTrigger**](#variable-scoretrigger)   = `ETryllScoreTrigger::Never`<br> |
|  FString | [**Statement**](#variable-statement)  <br> |
|  FString | [**SystemPrompt**](#variable-systemprompt)  <br> |
|  float | [**Threshold**](#variable-threshold)   = `0.5f`<br> |
|  FString | [**UncertainChoice**](#variable-uncertainchoice)  <br> |
|  bool | [**bOverrideSystemPrompt**](#variable-boverridesystemprompt)   = `false`<br> |












































## Detailed Description


One specialist question scored in the same batched call as its siblings. 


    
## Public Attributes Documentation




### variable BooleanTrigger 

```C++
ETryllBooleanTrigger FTryllClassifyAttributeDef::BooleanTrigger;
```



Boolean only. Which decided value takes triggered\_exit. 


        

<hr>



### variable Choices 

```C++
TArray<FTryllClassifyAttributeChoice> FTryllClassifyAttributeDef::Choices;
```



Choice and Score only. Choice order is arbitrary; Score order is lowest to highest. 2-26 entries. Forbidden for Boolean. 


        

<hr>



### variable Kind 

```C++
ETryllAttributeKind FTryllClassifyAttributeDef::Kind;
```



Choice writes the winning id. Boolean writes true/false for one statement. Score writes a 0..1 expected level. 


        

<hr>



### variable Margin 

```C++
float FTryllClassifyAttributeDef::Margin;
```



Choice only. Minimum gap between the top two probabilities. 


        

<hr>



### variable Name 

```C++
FString FTryllClassifyAttributeDef::Name;
```



Slot suffix and diagnostics key. Unique within the parent vector. Combined with the node's output\_name as &lt;output\_name&gt;\_&lt;name&gt;. 


        

<hr>



### variable ScoreThreshold 

```C++
float FTryllClassifyAttributeDef::ScoreThreshold;
```



Score only. Threshold in the normalised 0..1 range. 


        

<hr>



### variable ScoreTrigger 

```C++
ETryllScoreTrigger FTryllClassifyAttributeDef::ScoreTrigger;
```



Score only. Comparison of the normalised score with score\_threshold. 


        

<hr>



### variable Statement 

```C++
FString FTryllClassifyAttributeDef::Statement;
```



Boolean only. The proposition evaluated against the input. 


        

<hr>



### variable SystemPrompt 

```C++
FString FTryllClassifyAttributeDef::SystemPrompt;
```




<hr>



### variable Threshold 

```C++
float FTryllClassifyAttributeDef::Threshold;
```



Choice: minimum winning probability. Boolean: p\_true cutoff. 


        

<hr>



### variable UncertainChoice 

```C++
FString FTryllClassifyAttributeDef::UncertainChoice;
```



Choice only. Written when the winner is below threshold or margin. One of the choice ids, or the reserved "#uncertain". A choice id uses that choice's triggers flag, including when that id itself wins. "#uncertain" is not a menu letter and does not trigger. 


        

<hr>



### variable bOverrideSystemPrompt 

```C++
bool FTryllClassifyAttributeDef::bOverrideSystemPrompt;
```



Specialist Mustache template. Empty selects a kind-specific default. Choice and Score must reference intents\_block. Boolean must reference statement; the node appends the fixed True/False menu. Set to true to override the inherited SystemPrompt value. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllClassifyAttributeDef.h`

