

# Class UTryllNodeParamsBase



[**ClassList**](annotated.md) **>** [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md)



[More...](#detailed-description)

* `#include <TryllNodeParamsBase.h>`



Inherits the following classes: UObject


Inherited by the following classes: [UTryllCannedResponseParams](class_u_tryll_canned_response_params.md),  [UTryllClassifyIntentLLMParams](class_u_tryll_classify_intent_l_l_m_params.md),  [UTryllClassifyIntentParams](class_u_tryll_classify_intent_params.md),  [UTryllGenerateAndSpeakParams](class_u_tryll_generate_and_speak_params.md),  [UTryllGenerateParams](class_u_tryll_generate_params.md),  [UTryllHumanMessageGuardrailParams](class_u_tryll_human_message_guardrail_params.md),  [UTryllInstructionParams](class_u_tryll_instruction_params.md),  [UTryllIntentToInstructionParams](class_u_tryll_intent_to_instruction_params.md),  [UTryllRetrieveParams](class_u_tryll_retrieve_params.md),  [UTryllSpeakParams](class_u_tryll_speak_params.md),  [UTryllToolCallParams](class_u_tryll_tool_call_params.md)
































## Public Functions

| Type | Name |
| ---: | :--- |
| virtual TArray&lt; FString &gt; | [**GetExitTargetOptions**](#function-getexittargetoptions) () const<br> |
| virtual ETryllNodeType | [**GetNodeType**](#function-getnodetype) () const<br> |
| virtual flatbuffers::Offset&lt; void &gt; | [**Pack**](#function-pack) (flatbuffers::FlatBufferBuilder & Fbb, ::Tryll::NodeParams::NodeParams & OutType) const<br> |
|  ETryllNodeType return | [**static\_cast**](#function-static_cast) (0) <br> |




























## Detailed Description


Abstract base for typed node-parameter UObjects (generated subclasses in this directory). 


    
## Public Functions Documentation




### function GetExitTargetOptions 

```C++
virtual TArray< FString > UTryllNodeParamsBase::GetExitTargetOptions () const
```



Dropdown options for fields tagged with FlatBuffers `(exit)`. Walks the Outer chain to find the enclosing [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) (typically owned by a [**UTryllWorkflowAsset**](class_u_tryll_workflow_asset.md) or [**UTryllAgentComponent**](class_u_tryll_agent_component.md)), then returns: [""] + node names from that graph.


The leading empty string represents END (turn finishes after this node); Unreal's Details panel renders it as a blank entry in the dropdown.


Bound via UPROPERTY meta=(GetOptions="GetExitTargetOptions") on every codegen-emitted exit field. When the params UObject is not part of an editable graph (orphaned / runtime-constructed), returns just [""]. 


        

<hr>



### function GetNodeType 

```C++
virtual ETryllNodeType UTryllNodeParamsBase::GetNodeType () const
```




<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllNodeParamsBase::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    ::Tryll::NodeParams::NodeParams & OutType
) const
```




<hr>



### function static\_cast 

```C++
ETryllNodeType return UTryllNodeParamsBase::static_cast (
    0
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllNodeParamsBase.h`

