

# Struct FTryllWorkflowModelRefs



[**ClassList**](annotated.md) **>** [**FTryllWorkflowModelRefs**](struct_f_tryll_workflow_model_refs.md)












































## Public Static Functions

| Type | Name |
| ---: | :--- |
|  TArray&lt; [**FTryllWorkflowModelRef**](struct_f_tryll_workflow_model_ref.md) &gt; | [**Collect**](#function-collect) (const [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) & Graph) <br> |
|  TArray&lt; [**FTryllWorkflowModelRef**](struct_f_tryll_workflow_model_ref.md) &gt; | [**CollectVoice**](#function-collectvoice) (const [**FTryllVoiceProfileData**](struct_f_tryll_voice_profile_data.md) & Voice) <br> |
|  TArray&lt; [**FTryllWorkflowModelViolation**](struct_f_tryll_workflow_model_violation.md) &gt; | [**FindNonProductionRefs**](#function-findnonproductionrefs) (const [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) & Graph, const [**UTryllModelManifest**](class_u_tryll_model_manifest.md) \* Manifest) <br> |
|  TArray&lt; [**FTryllWorkflowModelViolation**](struct_f_tryll_workflow_model_violation.md) &gt; | [**FindNonProductionVoiceRefs**](#function-findnonproductionvoicerefs) (const [**FTryllVoiceProfileData**](struct_f_tryll_voice_profile_data.md) & Voice, const [**UTryllModelManifest**](class_u_tryll_model_manifest.md) \* Manifest) <br> |
|  bool | [**NeedsProductionVad**](#function-needsproductionvad) (const TArray&lt; [**FTryllWorkflowModelRef**](struct_f_tryll_workflow_model_ref.md) &gt; & Refs, const [**UTryllModelManifest**](class_u_tryll_model_manifest.md) \* Manifest, FString & OutReason) <br> |


























## Public Static Functions Documentation




### function Collect 

```C++
static TArray< FTryllWorkflowModelRef > FTryllWorkflowModelRefs::Collect (
    const FTryllGraphDescription & Graph
) 
```



Every non-empty [**FTryllModelName**](struct_f_tryll_model_name.md) field tagged meta=(TryllModelType=...) that is active (bOverride\* true, or no override toggle). 


        

<hr>



### function CollectVoice 

```C++
static TArray< FTryllWorkflowModelRef > FTryllWorkflowModelRefs::CollectVoice (
    const FTryllVoiceProfileData & Voice
) 
```



The voice profile's model reference (TtsModelName), through the same tagged-field scan. Empty when unset. 


        

<hr>



### function FindNonProductionRefs 

```C++
static TArray< FTryllWorkflowModelViolation > FTryllWorkflowModelRefs::FindNonProductionRefs (
    const FTryllGraphDescription & Graph,
    const UTryllModelManifest * Manifest
) 
```



Refs whose registration is not Production (including unlisted → None). 


        

<hr>



### function FindNonProductionVoiceRefs 

```C++
static TArray< FTryllWorkflowModelViolation > FTryllWorkflowModelRefs::FindNonProductionVoiceRefs (
    const FTryllVoiceProfileData & Voice,
    const UTryllModelManifest * Manifest
) 
```



Voice-profile refs whose registration is not Production. 


        

<hr>



### function NeedsProductionVad 

```C++
static bool FTryllWorkflowModelRefs::NeedsProductionVad (
    const TArray< FTryllWorkflowModelRef > & Refs,
    const UTryllModelManifest * Manifest,
    FString & OutReason
) 
```



True when any STT model is referenced but the manifest has no Production VAD. Offline STT needs the server's default\_vad\_model staged into the build. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllWorkflowModelRefs.h`

