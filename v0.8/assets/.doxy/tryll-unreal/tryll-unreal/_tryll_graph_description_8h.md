

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
| enum uint8 | [**ETryllBranchTest**](#enum-etryllbranchtest)  <br> |
| enum uint8 | [**ETryllCancelMode**](#enum-etryllcancelmode)  <br> |
| enum uint8 | [**ETryllCannedResponseSelectionStrategy**](#enum-etryllcannedresponseselectionstrategy)  <br> |
| enum uint8 | [**ETryllHistoryRole**](#enum-etryllhistoryrole)  <br> |
| enum uint8 | [**ETryllInferenceEngine**](#enum-etryllinferenceengine)  <br> |
| enum uint8 | [**ETryllKnowledgeSourceKind**](#enum-etryllknowledgesourcekind)  <br> |
| enum uint8 | [**ETryllKvCacheType**](#enum-etryllkvcachetype)  <br> |
| enum uint8 | [**ETryllNodeType**](#enum-etryllnodetype)  <br> |
| enum uint8 | [**ETryllNotifyClient**](#enum-etryllnotifyclient)  <br> |
| enum uint8 | [**ETryllPlacement**](#enum-etryllplacement)  <br> |
| enum uint8 | [**ETryllRetrievalMode**](#enum-etryllretrievalmode)  <br> |
| enum uint8 | [**ETryllSendAnswer**](#enum-etryllsendanswer)  <br> |
| enum uint8 | [**ETryllToolCallDisposition**](#enum-etrylltoolcalldisposition)  <br> |
| enum uint8 | [**ETryllToolCallMode**](#enum-etrylltoolcallmode)  <br> |
| enum uint8 | [**ETryllTurnStatus**](#enum-etryllturnstatus)  <br> |
















































## Public Types Documentation




### enum ETryllBranchTest 

```C++
enum ETryllBranchTest {
    NonEmpty = 0,
    IsTrue = 1,
    Equals = 2,
    EqualsIgnoreCase = 3,
    Contains = 4,
    Regex = 5,
    Expression = 6
};
```



Predicate policy for the Branch node. Mirrors the BranchTest enum in common\_enums.fbs — ordinals must stay in sync. 


        

<hr>



### enum ETryllCancelMode 

```C++
enum ETryllCancelMode {
    StopAndKeep = 0,
    StopAndDiscard = 1
};
```



How a mid-turn cancel treats the in-flight interaction. Mirrors CancelMode in messages.fbs. 


        

<hr>



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



### enum ETryllHistoryRole 

```C++
enum ETryllHistoryRole {
    None = 0,
    Assistant = 1
};
```



Controls whether/how a node's slot replays as history on later turns. Mirrors HistoryRole in common\_enums.fbs. 


        

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
    SherpaOnnx = 6,
    OnnxRuntime = 7
};
```



Inference engine the session should use for model loading. Mirrors the FlatBuffers Engine::InferenceEngine enum in messages.fbs — ordinal values must stay in sync. 


        

<hr>



### enum ETryllKnowledgeSourceKind 

```C++
enum ETryllKnowledgeSourceKind {
    FromFile = 0,
    FromName = 1
};
```



How Retrieve addresses a knowledge source. Mirrors KnowledgeSourceKind in common\_enums.fbs — ordinals must stay in sync. 


        

<hr>



### enum ETryllKvCacheType 

```C++
enum ETryllKvCacheType {
    Inherit = 0,
    F16 = 1,
    Q8_0 = 2,
    Q4_0 = 3
};
```



llama.cpp KV cache dtype on ContextOverrides. Mirrors KvCacheType in common\_enums.fbs — ordinals must stay in sync. 


        

<hr>



### enum ETryllNodeType 

```C++
enum ETryllNodeType {
    Generate = 0,
    RegexGuardrail = 1,
    CannedResponse = 2,
    ToolCall = 3,
    Retrieve = 4,
    Instruction = 5,
    ClassifyIntent = 6,
    IntentToInstruction = 7,
    ClassifyIntentLLM = 8,
    GenerateAndSpeak = 9,
    Speak = 10,
    Pause = 11,
    Transform = 12,
    Branch = 13,
    WriteToMemory = 14,
    Summarize = 15
};
```



Workflow node types supported by the server. Mirrors the NodeParams union order in nodes.fbs (0-based; wire discriminator is +1). 


        

<hr>



### enum ETryllNotifyClient 

```C++
enum ETryllNotifyClient {
    Disabled = 0,
    OnFound = 1,
    Always = 2
};
```



When a ClassifyIntent / ClassifyIntentLLM node emits a NodeEvent. Mirrors NotifyClient in common\_enums.fbs — ordinals must stay in sync. 


        

<hr>



### enum ETryllPlacement 

```C++
enum ETryllPlacement {
    BeforeUserAsUser = 0,
    BeforeUserAsSystem = 1,
    AfterUserAsUser = 2,
    AfterUserAsSystem = 3
};
```



Where the rendered Mustache body is inserted relative to the node's resolved Input. Mirrors the Placement enum in common\_enums.fbs — ordinals must stay in sync. There is no more InPlaceOfUser — use a Transform node + Input to fully replace what the model sees. 


        

<hr>



### enum ETryllRetrievalMode 

```C++
enum ETryllRetrievalMode {
    Dense = 0,
    Lexical = 1,
    Hybrid = 2
};
```



RetrieveNode search path. Mirrors the RetrievalMode enum in common\_enums.fbs — ordinals must stay in sync. 


        

<hr>



### enum ETryllSendAnswer 

```C++
enum ETryllSendAnswer {
    None = 0,
    Whole = 1,
    Streamed = 2
};
```



Controls answer-text delivery to the client. Mirrors SendAnswer in common\_enums.fbs. 


        

<hr>



### enum ETryllToolCallDisposition 

```C++
enum ETryllToolCallDisposition {
    RouteOnly = 0,
    Acknowledge = 1,
    Notify = 2,
    NotifyAndAcknowledge = 3,
    Pause = 4,
    PauseAndAcknowledge = 5,
    AwaitResult = 6
};
```



End-to-end ToolCall notify / pause / history policy. Mirrors ToolCallDisposition in common\_enums.fbs — ordinals must stay in sync. 


        

<hr>



### enum ETryllToolCallMode 

```C++
enum ETryllToolCallMode {
    DetectOnly = 0,
    CallOrAnswer = 1,
    RequireCall = 2
};
```



Tool-choice policy for the ToolCall node. Mirrors the ToolCallMode enum in common\_enums.fbs — ordinals must stay in sync. 


        

<hr>



### enum ETryllTurnStatus 

```C++
enum ETryllTurnStatus {
    Success = 0,
    Error = 1,
    Cancelled = 2,
    PauseTimedOut = 3
};
```



Outcome of a workflow turn. Mirrors TurnStatus in messages.fbs. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllGraphDescription.h`

