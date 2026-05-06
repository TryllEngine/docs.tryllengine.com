

# File TryllGraphDescription.h



[**FileList**](files.md) **>** [**client-unreal**](dir_95d666eee9112f2bfa7f2b736b6243b9.md) **>** [**Source**](dir_e37f8a870dc803113e92bf135247a735.md) **>** [**TryllClient**](dir_0869abba98a308e3c3eadd7e169e0f62.md) **>** [**Public**](dir_338741d27b4bda5805009de80ddaf6fc.md) **>** [**TryllGraphDescription.h**](_tryll_graph_description_8h.md)





* `#include "CoreMinimal.h"`
* `#include "TryllGraphDescription.generated.h"`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTryllExitRoute**](struct_f_tryll_exit_route.md) <br> |
| struct | [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) <br> |
| struct | [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) <br> |
| struct | [**FTryllNodeDescription**](struct_f_tryll_node_description.md) <br> |
| struct | [**FTryllNodeParam**](struct_f_tryll_node_param.md) <br> |
| struct | [**FTryllToolDefinition**](struct_f_tryll_tool_definition.md) <br> |
| struct | [**FTryllToolParamDefinition**](struct_f_tryll_tool_param_definition.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| enum uint8 | [**ETryllInferenceEngine**](#enum-etryllinferenceengine)  <br> |
| enum uint8 | [**ETryllNodeType**](#enum-etryllnodetype)  <br> |
| enum uint8 | [**ETryllTurnStatus**](#enum-etryllturnstatus)  <br> |
















































## Public Types Documentation




### enum ETryllInferenceEngine 

```C++
enum ETryllInferenceEngine {
    Mock = 0,
    LlamaCpp = 1,
    OnnxGenAI = 2,
    WindowsML = 3,
    OpenVino = 4,
    TensorRtLlm = 5
};
```



Inference engine the session should use for model loading. Mirrors the FlatBuffers InferenceEngine enum in messages.fbs — ordinal values must stay in sync. 


        

<hr>



### enum ETryllNodeType 

```C++
enum ETryllNodeType {
    Generate = 0,
    HumanMessageGuardrail = 1,
    CannedResponse = 2,
    ToolCall = 3,
    Retrieve = 4,
    Instruction = 5
};
```



Workflow node types supported by the server. Mirrors the FlatBuffers NodeType enum in messages.fbs — ordinal values must stay in sync. 


        

<hr>



### enum ETryllTurnStatus 

```C++
enum ETryllTurnStatus {
    Success = 0,
    Error = 1,
    Cancelled = 2
};
```



Outcome of a workflow turn. Mirrors TurnStatus in messages.fbs. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo/server/client-unreal/Source/TryllClient/Public/TryllGraphDescription.h`

