

# File TryllGraphBuilder.Nodes.h



[**FileList**](files.md) **>** [**client-unreal**](dir_95d666eee9112f2bfa7f2b736b6243b9.md) **>** [**Source**](dir_e37f8a870dc803113e92bf135247a735.md) **>** [**TryllClient**](dir_0869abba98a308e3c3eadd7e169e0f62.md) **>** [**Public**](dir_338741d27b4bda5805009de80ddaf6fc.md) **>** [**Generated**](dir_1665866c692210bc175b708647b46bfb.md) **>** [**TryllGraphBuilder.Nodes.h**](_tryll_graph_builder_8_nodes_8h.md)





* `#include "CoreMinimal.h"`
* `#include "TryllGraphDescription.h"`
* `#include "Generated/Nodes/TryllGenerateParams.h"`
* `#include "Generated/Nodes/TryllHumanMessageGuardrailParams.h"`
* `#include "Generated/Nodes/TryllCannedResponseParams.h"`
* `#include "Generated/Nodes/TryllToolCallParams.h"`
* `#include "Generated/Nodes/TryllRetrieveParams.h"`
* `#include "Generated/Nodes/TryllInstructionParams.h"`
* `#include "Generated/Nodes/TryllClassifyIntentParams.h"`
* `#include "Generated/Nodes/TryllClassifyIntentLLMParams.h"`
* `#include "Generated/Nodes/TryllIntentToInstructionParams.h"`
* `#include "CodegenFingerprint.gen.h"`





































## Public Functions

| Type | Name |
| ---: | :--- |
|  TRYLLCLIENT\_API [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddCannedResponseNode**](#function-addcannedresponsenode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllCannedResponseParams**](class_u_tryll_canned_response_params.md) \* Params) <br> |
|  TRYLLCLIENT\_API [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddClassifyIntentLLMNode**](#function-addclassifyintentllmnode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllClassifyIntentLLMParams**](class_u_tryll_classify_intent_l_l_m_params.md) \* Params) <br> |
|  TRYLLCLIENT\_API [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddClassifyIntentNode**](#function-addclassifyintentnode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllClassifyIntentParams**](class_u_tryll_classify_intent_params.md) \* Params) <br> |
|  TRYLLCLIENT\_API [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddGenerateNode**](#function-addgeneratenode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllGenerateParams**](class_u_tryll_generate_params.md) \* Params) <br> |
|  TRYLLCLIENT\_API [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddHumanMessageGuardrailNode**](#function-addhumanmessageguardrailnode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllHumanMessageGuardrailParams**](class_u_tryll_human_message_guardrail_params.md) \* Params) <br> |
|  TRYLLCLIENT\_API [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddInstructionNode**](#function-addinstructionnode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllInstructionParams**](class_u_tryll_instruction_params.md) \* Params) <br> |
|  TRYLLCLIENT\_API [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddIntentToInstructionNode**](#function-addintenttoinstructionnode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllIntentToInstructionParams**](class_u_tryll_intent_to_instruction_params.md) \* Params) <br> |
|  TRYLLCLIENT\_API [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddRetrieveNode**](#function-addretrievenode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllRetrieveParams**](class_u_tryll_retrieve_params.md) \* Params) <br> |
|  TRYLLCLIENT\_API [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & | [**AddToolCallNode**](#function-addtoolcallnode) ([**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) & Builder, FString Name, [**UTryllToolCallParams**](class_u_tryll_tool_call_params.md) \* Params) <br> |




























## Public Functions Documentation




### function AddCannedResponseNode 

```C++
TRYLLCLIENT_API FTryllGraphBuilder & AddCannedResponseNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllCannedResponseParams * Params
) 
```



Add a CannedResponse node to the graph. 


        

<hr>



### function AddClassifyIntentLLMNode 

```C++
TRYLLCLIENT_API FTryllGraphBuilder & AddClassifyIntentLLMNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllClassifyIntentLLMParams * Params
) 
```



Add a ClassifyIntentLLM node to the graph. 


        

<hr>



### function AddClassifyIntentNode 

```C++
TRYLLCLIENT_API FTryllGraphBuilder & AddClassifyIntentNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllClassifyIntentParams * Params
) 
```



Add a ClassifyIntent node to the graph. 


        

<hr>



### function AddGenerateNode 

```C++
TRYLLCLIENT_API FTryllGraphBuilder & AddGenerateNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllGenerateParams * Params
) 
```



Generated typed AddX builder methods mixed into [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md). Include this header in TryllGraphDescription.h. Add a Generate node to the graph. 


        

<hr>



### function AddHumanMessageGuardrailNode 

```C++
TRYLLCLIENT_API FTryllGraphBuilder & AddHumanMessageGuardrailNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllHumanMessageGuardrailParams * Params
) 
```



Add a HumanMessageGuardrail node to the graph. 


        

<hr>



### function AddInstructionNode 

```C++
TRYLLCLIENT_API FTryllGraphBuilder & AddInstructionNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllInstructionParams * Params
) 
```



Add a Instruction node to the graph. 


        

<hr>



### function AddIntentToInstructionNode 

```C++
TRYLLCLIENT_API FTryllGraphBuilder & AddIntentToInstructionNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllIntentToInstructionParams * Params
) 
```



Add a IntentToInstruction node to the graph. 


        

<hr>



### function AddRetrieveNode 

```C++
TRYLLCLIENT_API FTryllGraphBuilder & AddRetrieveNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllRetrieveParams * Params
) 
```



Add a Retrieve node to the graph. 


        

<hr>



### function AddToolCallNode 

```C++
TRYLLCLIENT_API FTryllGraphBuilder & AddToolCallNode (
    FTryllGraphBuilder & Builder,
    FString Name,
    UTryllToolCallParams * Params
) 
```



Add a ToolCall node to the graph. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/Generated/TryllGraphBuilder.Nodes.h`

