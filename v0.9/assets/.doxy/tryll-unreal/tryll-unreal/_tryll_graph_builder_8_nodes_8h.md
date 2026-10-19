

# File TryllGraphBuilder.Nodes.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**Generated**](dir_2ea38e7786b58b4326da8aed9160f931.md) **>** [**TryllGraphBuilder.Nodes.h**](_tryll_graph_builder_8_nodes_8h.md)





* `#include "CoreMinimal.h"`
* `#include "TryllGraphDescription.h"`
* `#include "Generated/Nodes/TryllGenerateParams.h"`
* `#include "Generated/Nodes/TryllRegexGuardrailParams.h"`
* `#include "Generated/Nodes/TryllCannedResponseParams.h"`
* `#include "Generated/Nodes/TryllToolCallParams.h"`
* `#include "Generated/Nodes/TryllRetrieveParams.h"`
* `#include "Generated/Nodes/TryllInstructionParams.h"`
* `#include "Generated/Nodes/TryllClassifyIntentParams.h"`
* `#include "Generated/Nodes/TryllClassifyIntentLLMParams.h"`
* `#include "Generated/Nodes/TryllIntentToInstructionParams.h"`
* `#include "Generated/Nodes/TryllGenerateAndSpeakParams.h"`
* `#include "Generated/Nodes/TryllSpeakParams.h"`
* `#include "Generated/Nodes/TryllPauseParams.h"`
* `#include "Generated/Nodes/TryllTransformParams.h"`
* `#include "Generated/Nodes/TryllBranchParams.h"`
* `#include "Generated/Nodes/TryllWriteToMemoryParams.h"`
* `#include "Generated/Nodes/TryllSummarizeParams.h"`
* `#include "Generated/Nodes/TryllClassifyAttributesLLMParams.h"`
* `#include "CodegenFingerprint.gen.h"`
* `#include <string_view>`





































## Public Functions

| Type | Name |
| ---: | :--- |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddBranchNode**](#function-addbranchnode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllBranchParams**](class_u_tryll_branch_params.md) \* Params) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddCannedResponseNode**](#function-addcannedresponsenode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllCannedResponseParams**](class_u_tryll_canned_response_params.md) \* Params) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddClassifyAttributesLLMNode**](#function-addclassifyattributesllmnode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllClassifyAttributesLLMParams**](class_u_tryll_classify_attributes_l_l_m_params.md) \* Params) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddClassifyIntentLLMNode**](#function-addclassifyintentllmnode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllClassifyIntentLLMParams**](class_u_tryll_classify_intent_l_l_m_params.md) \* Params) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddClassifyIntentNode**](#function-addclassifyintentnode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllClassifyIntentParams**](class_u_tryll_classify_intent_params.md) \* Params) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddGenerateAndSpeakNode**](#function-addgenerateandspeaknode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllGenerateAndSpeakParams**](class_u_tryll_generate_and_speak_params.md) \* Params) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddGenerateNode**](#function-addgeneratenode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllGenerateParams**](class_u_tryll_generate_params.md) \* Params) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddInstructionNode**](#function-addinstructionnode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllInstructionParams**](class_u_tryll_instruction_params.md) \* Params) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddIntentToInstructionNode**](#function-addintenttoinstructionnode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllIntentToInstructionParams**](class_u_tryll_intent_to_instruction_params.md) \* Params) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddPauseNode**](#function-addpausenode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllPauseParams**](class_u_tryll_pause_params.md) \* Params) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddRegexGuardrailNode**](#function-addregexguardrailnode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllRegexGuardrailParams**](class_u_tryll_regex_guardrail_params.md) \* Params) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddRetrieveNode**](#function-addretrievenode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllRetrieveParams**](class_u_tryll_retrieve_params.md) \* Params) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddSpeakNode**](#function-addspeaknode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllSpeakParams**](class_u_tryll_speak_params.md) \* Params) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddSummarizeNode**](#function-addsummarizenode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllSummarizeParams**](class_u_tryll_summarize_params.md) \* Params) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddToolCallNode**](#function-addtoolcallnode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllToolCallParams**](class_u_tryll_tool_call_params.md) \* Params) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddTransformNode**](#function-addtransformnode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllTransformParams**](class_u_tryll_transform_params.md) \* Params) <br> |
|  [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddWriteToMemoryNode**](#function-addwritetomemorynode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllWriteToMemoryParams**](class_u_tryll_write_to_memory_params.md) \* Params) <br> |




























## Public Functions Documentation




### function AddBranchNode 

```C++
FTryllGraphBuilder & AddBranchNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllBranchParams * Params
) 
```



Add a Branch node to the graph. 


        

<hr>



### function AddCannedResponseNode 

```C++
FTryllGraphBuilder & AddCannedResponseNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllCannedResponseParams * Params
) 
```



Add a CannedResponse node to the graph. 


        

<hr>



### function AddClassifyAttributesLLMNode 

```C++
FTryllGraphBuilder & AddClassifyAttributesLLMNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllClassifyAttributesLLMParams * Params
) 
```



Add a ClassifyAttributesLLM node to the graph. 


        

<hr>



### function AddClassifyIntentLLMNode 

```C++
FTryllGraphBuilder & AddClassifyIntentLLMNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllClassifyIntentLLMParams * Params
) 
```



Add a ClassifyIntentLLM node to the graph. 


        

<hr>



### function AddClassifyIntentNode 

```C++
FTryllGraphBuilder & AddClassifyIntentNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllClassifyIntentParams * Params
) 
```



Add a ClassifyIntent node to the graph. 


        

<hr>



### function AddGenerateAndSpeakNode 

```C++
FTryllGraphBuilder & AddGenerateAndSpeakNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllGenerateAndSpeakParams * Params
) 
```



Add a GenerateAndSpeak node to the graph. 


        

<hr>



### function AddGenerateNode 

```C++
FTryllGraphBuilder & AddGenerateNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllGenerateParams * Params
) 
```



Generated typed AddX builder methods mixed into [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md). Include this header in TryllGraphDescription.h. Add a Generate node to the graph. 


        

<hr>



### function AddInstructionNode 

```C++
FTryllGraphBuilder & AddInstructionNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllInstructionParams * Params
) 
```



Add a Instruction node to the graph. 


        

<hr>



### function AddIntentToInstructionNode 

```C++
FTryllGraphBuilder & AddIntentToInstructionNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllIntentToInstructionParams * Params
) 
```



Add a IntentToInstruction node to the graph. 


        

<hr>



### function AddPauseNode 

```C++
FTryllGraphBuilder & AddPauseNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllPauseParams * Params
) 
```



Add a Pause node to the graph. 


        

<hr>



### function AddRegexGuardrailNode 

```C++
FTryllGraphBuilder & AddRegexGuardrailNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllRegexGuardrailParams * Params
) 
```



Add a RegexGuardrail node to the graph. 


        

<hr>



### function AddRetrieveNode 

```C++
FTryllGraphBuilder & AddRetrieveNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllRetrieveParams * Params
) 
```



Add a Retrieve node to the graph. 


        

<hr>



### function AddSpeakNode 

```C++
FTryllGraphBuilder & AddSpeakNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllSpeakParams * Params
) 
```



Add a Speak node to the graph. 


        

<hr>



### function AddSummarizeNode 

```C++
FTryllGraphBuilder & AddSummarizeNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllSummarizeParams * Params
) 
```



Add a Summarize node to the graph. 


        

<hr>



### function AddToolCallNode 

```C++
FTryllGraphBuilder & AddToolCallNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllToolCallParams * Params
) 
```



Add a ToolCall node to the graph. 


        

<hr>



### function AddTransformNode 

```C++
FTryllGraphBuilder & AddTransformNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllTransformParams * Params
) 
```



Add a Transform node to the graph. 


        

<hr>



### function AddWriteToMemoryNode 

```C++
FTryllGraphBuilder & AddWriteToMemoryNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllWriteToMemoryParams * Params
) 
```



Add a WriteToMemory node to the graph. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/TryllGraphBuilder.Nodes.h`

