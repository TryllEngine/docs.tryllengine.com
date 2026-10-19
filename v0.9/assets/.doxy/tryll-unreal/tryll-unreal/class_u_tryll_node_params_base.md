

# Class UTryllNodeParamsBase



[**ClassList**](annotated.md) **>** [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md)



[More...](#detailed-description)

* `#include <TryllNodeParamsBase.h>`



Inherits the following classes: UObject


Inherited by the following classes: [UTryllBranchParams](class_u_tryll_branch_params.md),  [UTryllCannedResponseParams](class_u_tryll_canned_response_params.md),  [UTryllClassifyAttributesLLMParams](class_u_tryll_classify_attributes_l_l_m_params.md),  [UTryllClassifyIntentLLMParams](class_u_tryll_classify_intent_l_l_m_params.md),  [UTryllClassifyIntentParams](class_u_tryll_classify_intent_params.md),  [UTryllGenerateAndSpeakParams](class_u_tryll_generate_and_speak_params.md),  [UTryllGenerateParams](class_u_tryll_generate_params.md),  [UTryllInstructionParams](class_u_tryll_instruction_params.md),  [UTryllIntentToInstructionParams](class_u_tryll_intent_to_instruction_params.md),  [UTryllPauseParams](class_u_tryll_pause_params.md),  [UTryllRegexGuardrailParams](class_u_tryll_regex_guardrail_params.md),  [UTryllRetrieveParams](class_u_tryll_retrieve_params.md),  [UTryllSpeakParams](class_u_tryll_speak_params.md),  [UTryllSummarizeParams](class_u_tryll_summarize_params.md),  [UTryllToolCallParams](class_u_tryll_tool_call_params.md),  [UTryllTransformParams](class_u_tryll_transform_params.md),  [UTryllWriteToMemoryParams](class_u_tryll_write_to_memory_params.md)
































## Public Functions

| Type | Name |
| ---: | :--- |
| virtual TArray&lt; FString &gt; | [**GetExitTargetOptions**](#function-getexittargetoptions) () const<br> |
| virtual ETryllNodeType | [**GetNodeType**](#function-getnodetype) () const<br> |
|  FString | [**GetOwnerNodeName**](#function-getownernodename) () const<br> |
| virtual bool | [**IsTtsConsumer**](#function-isttsconsumer) () const<br> |
| virtual flatbuffers::Offset&lt; void &gt; | [**Pack**](#function-pack) (flatbuffers::FlatBufferBuilder & Fbb, ::Tryll::NodeParams::NodeParams & OutType) const<br> |
| virtual void | [**PostLoad**](#function-postload) () override<br> |
| virtual void | [**Serialize**](#function-serialize) (FArchive & Ar) override<br> |
|  ETryllNodeType return | [**static\_cast**](#function-static_cast) (0) <br> |


## Public Static Functions

| Type | Name |
| ---: | :--- |
|  const [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) \* | [**FindEnclosingGraph**](#function-findenclosinggraph) (const UObject \* Obj) <br> |






















## Protected Functions

| Type | Name |
| ---: | :--- |
| virtual void | [**ApplyOldDefaults**](#function-applyolddefaults) (int32 SavedVersion) <br> |
| virtual bool | [**UpgradeFrom**](#function-upgradefrom) (int32 SavedVersion) <br> |




## Detailed Description


Abstract base for typed node-parameter UObjects (generated subclasses in this directory). 


    
## Public Functions Documentation




### function GetExitTargetOptions 

```C++
virtual TArray< FString > UTryllNodeParamsBase::GetExitTargetOptions () const
```



Dropdown options for fields tagged with FlatBuffers `(exit)`. Walks the Outer chain to find the enclosing [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) (typically owned by a [**UTryllWorkflowAsset**](class_u_tryll_workflow_asset.md)), then returns: [""] + node names from that graph.


The leading empty string represents END (turn finishes after this node); Unreal's Details panel renders it as a blank entry in the dropdown.


Bound via UPROPERTY meta=(GetOptions="GetExitTargetOptions") on every codegen-emitted exit field. When the params UObject is not part of an editable graph (orphaned / runtime-constructed), returns just [""]. 


        

<hr>



### function GetNodeType 

```C++
virtual ETryllNodeType UTryllNodeParamsBase::GetNodeType () const
```




<hr>



### function GetOwnerNodeName 

```C++
FString UTryllNodeParamsBase::GetOwnerNodeName () const
```



The node name that owns this Params instance within FindEnclosingGraph(this) — i.e. the [**FTryllNodeDescription**](struct_f_tryll_node_description.md) whose Params pointer equals `this` — or the empty string when not found. 


        

<hr>



### function IsTtsConsumer 

```C++
inline virtual bool UTryllNodeParamsBase::IsTtsConsumer () const
```



True when this node speaks through the agent's voice (Speak, GenerateAndSpeak). A graph with such a node needs a voice profile. From the schema's (tts\_consumer). 


        

<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllNodeParamsBase::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    ::Tryll::NodeParams::NodeParams & OutType
) const
```




<hr>



### function PostLoad 

```C++
virtual void UTryllNodeParamsBase::PostLoad () override
```




<hr>



### function Serialize 

```C++
virtual void UTryllNodeParamsBase::Serialize (
    FArchive & Ar
) override
```




<hr>



### function static\_cast 

```C++
ETryllNodeType return UTryllNodeParamsBase::static_cast (
    0
) 
```




<hr>
## Public Static Functions Documentation




### function FindEnclosingGraph 

```C++
static const FTryllGraphDescription * UTryllNodeParamsBase::FindEnclosingGraph (
    const UObject * Obj
) 
```



Walks the Outer chain from `Obj` to locate the enclosing [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) (a [**UTryllWorkflowAsset**](class_u_tryll_workflow_asset.md)'s Graph). Returns nullptr when `Obj` is not part of an editable graph (orphaned / runtime-constructed). Shared by GetExitTargetOptions and the editor-only slot-input dropdown / rename-propagation tooling (TryllClientEditor). 


        

<hr>
## Protected Functions Documentation




### function ApplyOldDefaults 

```C++
virtual void UTryllNodeParamsBase::ApplyOldDefaults (
    int32 SavedVersion
) 
```



Called before saved properties load, for packages saved at SavedVersion (an [**FTryllAssetVersion**](struct_f_tryll_asset_version.md)). 


        

<hr>



### function UpgradeFrom 

```C++
virtual bool UTryllNodeParamsBase::UpgradeFrom (
    int32 SavedVersion
) 
```



Called after load, for packages saved at SavedVersion. Returns true if anything changed. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllNodeParamsBase.h`

