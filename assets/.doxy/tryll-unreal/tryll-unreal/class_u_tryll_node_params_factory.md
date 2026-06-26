

# Class UTryllNodeParamsFactory



[**ClassList**](annotated.md) **>** [**UTryllNodeParamsFactory**](class_u_tryll_node_params_factory.md)



[More...](#detailed-description)

* `#include <TryllNodeParamsFactory.h>`



Inherits the following classes: UBlueprintFunctionLibrary




































## Public Static Functions

| Type | Name |
| ---: | :--- |
|  [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* | [**CloneParams**](#function-cloneparams) ([**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \* Source, UObject \* WorldContextObject) <br> |
|  [**UTryllCannedResponseParams**](class_u_tryll_canned_response_params.md) \* | [**MakeCannedResponseParams**](#function-makecannedresponseparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllClassifyIntentLLMParams**](class_u_tryll_classify_intent_l_l_m_params.md) \* | [**MakeClassifyIntentLLMParams**](#function-makeclassifyintentllmparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllClassifyIntentParams**](class_u_tryll_classify_intent_params.md) \* | [**MakeClassifyIntentParams**](#function-makeclassifyintentparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllGenerateAndSpeakParams**](class_u_tryll_generate_and_speak_params.md) \* | [**MakeGenerateAndSpeakParams**](#function-makegenerateandspeakparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllGenerateParams**](class_u_tryll_generate_params.md) \* | [**MakeGenerateParams**](#function-makegenerateparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllHumanMessageGuardrailParams**](class_u_tryll_human_message_guardrail_params.md) \* | [**MakeHumanMessageGuardrailParams**](#function-makehumanmessageguardrailparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllInstructionParams**](class_u_tryll_instruction_params.md) \* | [**MakeInstructionParams**](#function-makeinstructionparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllIntentToInstructionParams**](class_u_tryll_intent_to_instruction_params.md) \* | [**MakeIntentToInstructionParams**](#function-makeintenttoinstructionparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllRetrieveParams**](class_u_tryll_retrieve_params.md) \* | [**MakeRetrieveParams**](#function-makeretrieveparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllSpeakParams**](class_u_tryll_speak_params.md) \* | [**MakeSpeakParams**](#function-makespeakparams) (UObject \* WorldContextObject) <br> |
|  [**UTryllToolCallParams**](class_u_tryll_tool_call_params.md) \* | [**MakeToolCallParams**](#function-maketoolcallparams) (UObject \* WorldContextObject) <br> |


























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



### function MakeCannedResponseParams 

```C++
static UTryllCannedResponseParams * UTryllNodeParamsFactory::MakeCannedResponseParams (
    UObject * WorldContextObject
) 
```



Construct an authored CannedResponseParams with schema-default field values. 


        

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



### function MakeHumanMessageGuardrailParams 

```C++
static UTryllHumanMessageGuardrailParams * UTryllNodeParamsFactory::MakeHumanMessageGuardrailParams (
    UObject * WorldContextObject
) 
```



Construct an authored HumanMessageGuardrailParams with schema-default field values. 


        

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



### function MakeToolCallParams 

```C++
static UTryllToolCallParams * UTryllNodeParamsFactory::MakeToolCallParams (
    UObject * WorldContextObject
) 
```



Construct an authored ToolCallParams with schema-default field values. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/Generated/TryllNodeParamsFactory.h`

