

# Class UTryllNodeParamsFactory



[**ClassList**](annotated.md) **>** [**UTryllNodeParamsFactory**](class_u_tryll_node_params_factory.md)



[More...](#detailed-description)

* `#include <TryllNodeParamsFactory.h>`



Inherits the following classes: UBlueprintFunctionLibrary




































## Public Static Functions

| Type | Name |
| ---: | :--- |
|  [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* | [**CloneParams**](#function-cloneparams) ([**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* Source, UObject \* WorldContextObject) <br> |
|  [**UTryllBranchParams**](class_u_tryll_branch_params.md) \* | [**MakeBranchParams**](#function-makebranchparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllCannedResponseParams**](class_u_tryll_canned_response_params.md) \* | [**MakeCannedResponseParams**](#function-makecannedresponseparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllClassifyAttributesLLMParams**](class_u_tryll_classify_attributes_l_l_m_params.md) \* | [**MakeClassifyAttributesLLMParams**](#function-makeclassifyattributesllmparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllClassifyIntentLLMParams**](class_u_tryll_classify_intent_l_l_m_params.md) \* | [**MakeClassifyIntentLLMParams**](#function-makeclassifyintentllmparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllClassifyIntentParams**](class_u_tryll_classify_intent_params.md) \* | [**MakeClassifyIntentParams**](#function-makeclassifyintentparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllGenerateAndSpeakParams**](class_u_tryll_generate_and_speak_params.md) \* | [**MakeGenerateAndSpeakParams**](#function-makegenerateandspeakparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllGenerateParams**](class_u_tryll_generate_params.md) \* | [**MakeGenerateParams**](#function-makegenerateparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllInstructionParams**](class_u_tryll_instruction_params.md) \* | [**MakeInstructionParams**](#function-makeinstructionparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllIntentToInstructionParams**](class_u_tryll_intent_to_instruction_params.md) \* | [**MakeIntentToInstructionParams**](#function-makeintenttoinstructionparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllPauseParams**](class_u_tryll_pause_params.md) \* | [**MakePauseParams**](#function-makepauseparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllRegexGuardrailParams**](class_u_tryll_regex_guardrail_params.md) \* | [**MakeRegexGuardrailParams**](#function-makeregexguardrailparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllRetrieveParams**](class_u_tryll_retrieve_params.md) \* | [**MakeRetrieveParams**](#function-makeretrieveparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllSpeakParams**](class_u_tryll_speak_params.md) \* | [**MakeSpeakParams**](#function-makespeakparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllSummarizeParams**](class_u_tryll_summarize_params.md) \* | [**MakeSummarizeParams**](#function-makesummarizeparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllToolCallParams**](class_u_tryll_tool_call_params.md) \* | [**MakeToolCallParams**](#function-maketoolcallparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllTransformParams**](class_u_tryll_transform_params.md) \* | [**MakeTransformParams**](#function-maketransformparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllWriteToMemoryParams**](class_u_tryll_write_to_memory_params.md) \* | [**MakeWriteToMemoryParams**](#function-makewritetomemoryparams) (UObject \* WorldContextObject) <br> |


























## Detailed Description


Blueprint-callable factory for constructing concrete node params objects.


Allows Blueprints to create typed params for ChangeParams() and graph authoring without any per-project C++ glue.


Usage (Blueprint): MakeGenerateParams → set fields → Agent.ChangeParams(NodeName, Params) CloneParams(Baseline) → set one field → Agent.ChangeParams(NodeName, Cloned) 


    
## Public Static Functions Documentation




### function CloneParams 

```C++
static UTryllNodeParamsBase * UTryllNodeParamsFactory::CloneParams (
    UTryllNodeParamsBase * Source,
    UObject * WorldContextObject
) 
```



Deep-clone any [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) subclass. Typical mutation idiom: Clone = CloneParams(Agent-&gt;GetNodeParamsBaseline(NodeName)) // set fields on Clone Agent-&gt;ChangeParams(NodeName, Clone) 


        

<hr>



### function MakeBranchParams 

```C++
static UTryllBranchParams * UTryllNodeParamsFactory::MakeBranchParams (
    UObject * WorldContextObject
) 
```



Construct an authored BranchParams with schema-default field values. 


        

<hr>



### function MakeCannedResponseParams 

```C++
static UTryllCannedResponseParams * UTryllNodeParamsFactory::MakeCannedResponseParams (
    UObject * WorldContextObject
) 
```



Construct an authored CannedResponseParams with schema-default field values. 


        

<hr>



### function MakeClassifyAttributesLLMParams 

```C++
static UTryllClassifyAttributesLLMParams * UTryllNodeParamsFactory::MakeClassifyAttributesLLMParams (
    UObject * WorldContextObject
) 
```



Construct an authored ClassifyAttributesLLMParams with schema-default field values. 


        

<hr>



### function MakeClassifyIntentLLMParams 

```C++
static UTryllClassifyIntentLLMParams * UTryllNodeParamsFactory::MakeClassifyIntentLLMParams (
    UObject * WorldContextObject
) 
```



Construct an authored ClassifyIntentLLMParams with schema-default field values. 


        

<hr>



### function MakeClassifyIntentParams 

```C++
static UTryllClassifyIntentParams * UTryllNodeParamsFactory::MakeClassifyIntentParams (
    UObject * WorldContextObject
) 
```



Construct an authored ClassifyIntentParams with schema-default field values. 


        

<hr>



### function MakeGenerateAndSpeakParams 

```C++
static UTryllGenerateAndSpeakParams * UTryllNodeParamsFactory::MakeGenerateAndSpeakParams (
    UObject * WorldContextObject
) 
```



Construct an authored GenerateAndSpeakParams with schema-default field values. 


        

<hr>



### function MakeGenerateParams 

```C++
static UTryllGenerateParams * UTryllNodeParamsFactory::MakeGenerateParams (
    UObject * WorldContextObject
) 
```



Construct an authored GenerateParams with schema-default field values. 


        

<hr>



### function MakeInstructionParams 

```C++
static UTryllInstructionParams * UTryllNodeParamsFactory::MakeInstructionParams (
    UObject * WorldContextObject
) 
```



Construct an authored InstructionParams with schema-default field values. 


        

<hr>



### function MakeIntentToInstructionParams 

```C++
static UTryllIntentToInstructionParams * UTryllNodeParamsFactory::MakeIntentToInstructionParams (
    UObject * WorldContextObject
) 
```



Construct an authored IntentToInstructionParams with schema-default field values. 


        

<hr>



### function MakePauseParams 

```C++
static UTryllPauseParams * UTryllNodeParamsFactory::MakePauseParams (
    UObject * WorldContextObject
) 
```



Construct an authored PauseParams with schema-default field values. 


        

<hr>



### function MakeRegexGuardrailParams 

```C++
static UTryllRegexGuardrailParams * UTryllNodeParamsFactory::MakeRegexGuardrailParams (
    UObject * WorldContextObject
) 
```



Construct an authored RegexGuardrailParams with schema-default field values. 


        

<hr>



### function MakeRetrieveParams 

```C++
static UTryllRetrieveParams * UTryllNodeParamsFactory::MakeRetrieveParams (
    UObject * WorldContextObject
) 
```



Construct an authored RetrieveParams with schema-default field values. 


        

<hr>



### function MakeSpeakParams 

```C++
static UTryllSpeakParams * UTryllNodeParamsFactory::MakeSpeakParams (
    UObject * WorldContextObject
) 
```



Construct an authored SpeakParams with schema-default field values. 


        

<hr>



### function MakeSummarizeParams 

```C++
static UTryllSummarizeParams * UTryllNodeParamsFactory::MakeSummarizeParams (
    UObject * WorldContextObject
) 
```



Construct an authored SummarizeParams with schema-default field values. 


        

<hr>



### function MakeToolCallParams 

```C++
static UTryllToolCallParams * UTryllNodeParamsFactory::MakeToolCallParams (
    UObject * WorldContextObject
) 
```



Construct an authored ToolCallParams with schema-default field values. 


        

<hr>



### function MakeTransformParams 

```C++
static UTryllTransformParams * UTryllNodeParamsFactory::MakeTransformParams (
    UObject * WorldContextObject
) 
```



Construct an authored TransformParams with schema-default field values. 


        

<hr>



### function MakeWriteToMemoryParams 

```C++
static UTryllWriteToMemoryParams * UTryllNodeParamsFactory::MakeWriteToMemoryParams (
    UObject * WorldContextObject
) 
```



Construct an authored WriteToMemoryParams with schema-default field values. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/TryllNodeParamsFactory.h`

