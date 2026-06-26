

# File TryllGraphDescription.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllGraphDescription.h**](_tryll_graph_description_8h.md)





* `#include "CoreMinimal.h"`
* `#include "TryllModelName.h"`
* `#include "TryllGraphDescription.generated.h"`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTryllGraphBuilder**](struct_f_tryll_graph_builder.md) <br> |
| struct | [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) <br> |
| struct | [**FTryllGraphOptions**](struct_f_tryll_graph_options.md) <br> |
| struct | [**FTryllNodeDescription**](struct_f_tryll_node_description.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| enum uint8 | [**ETryllCannedResponseSelectionStrategy**](#enum-etryllcannedresponseselectionstrategy)  <br> |
| enum uint8 | [**ETryllInferenceEngine**](#enum-etryllinferenceengine)  <br> |
| enum uint8 | [**ETryllNodeType**](#enum-etryllnodetype)  <br> |
| enum uint8 | [**ETryllPlacement**](#enum-etryllplacement)  <br> |
| enum uint8 | [**ETryllTurnStatus**](#enum-etryllturnstatus)  <br> |
















































## Public Types Documentation




### enum ETryllCannedResponseSelectionStrategy 

```C++
enum ETryllCannedResponseSelectionStrategy {
    Random = 0,
    First = 1,
    RoundRobin = 2
};
```



Selection strategy for the CannedResponse node. Mirrors the CannedResponseSelectionStrategy enum in common\_enums.fbs. 


        

<hr>



### enum ETryllInferenceEngine 

```C++
enum ETryllInferenceEngine {
    Mock = 0,
    LlamaCpp = 1,
    OnnxGenAI = 2,
    WindowsML = 3,
    OpenVino = 4,
    TensorRtLlm = 5,
    SherpaOnnx = 6
};
```



Inference engine the session should use for model loading. Mirrors the FlatBuffers Engine::InferenceEngine enum in messages.fbs — ordinal values must stay in sync. 


        

<hr>



### enum ETryllNodeType 

```C++
enum ETryllNodeType {
    Generate = 0,
    HumanMessageGuardrail = 1,
    CannedResponse = 2,
    ToolCall = 3,
    Retrieve = 4,
    Instruction = 5,
    ClassifyIntent = 6,
    IntentToInstruction = 7,
    ClassifyIntentLLM = 8,
    GenerateAndSpeak = 9,
    Speak = 10
};
```



Workflow node types supported by the server. Mirrors the FlatBuffers NodeType enum in messages.fbs — ordinal values must stay in sync. 


        

<hr>



### enum ETryllPlacement 

```C++
enum ETryllPlacement {
    InPlaceOfUser = 0,
    BeforeUserAsUser = 1,
    BeforeUserAsSystem = 2,
    AfterUserAsUser = 3,
    AfterUserAsSystem = 4
};
```



Where the rendered Mustache body is inserted relative to the user turn. Mirrors the Placement enum in common\_enums.fbs — ordinals must stay in sync. 


        

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
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllGraphDescription.h`

