

# Struct FTryllVoiceProfileData



[**ClassList**](annotated.md) **>** [**FTryllVoiceProfileData**](struct_f_tryll_voice_profile_data.md)



[More...](#detailed-description)

* `#include <TryllVoiceProfile.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**DefaultDeliveryMode**](#variable-defaultdeliverymode)  <br> |
|  TArray&lt; [**FTryllDeliveryMode**](struct_f_tryll_delivery_mode.md) &gt; | [**DeliveryModes**](#variable-deliverymodes)  <br> |
|  float | [**Expressiveness**](#variable-expressiveness)   = `0.5f`<br> |
|  FString | [**Lang**](#variable-lang)  <br> |
|  ETryllVoicePreparation | [**Prepare**](#variable-prepare)   = `ETryllVoicePreparation::Eager`<br> |
|  [**FTryllStoragePath**](struct_f_tryll_storage_path.md) | [**ReferenceVoice**](#variable-referencevoice)  <br> |
|  int32 | [**SpeakerId**](#variable-speakerid)   = `0`<br> |
|  float | [**Speed**](#variable-speed)   = `1.f`<br> |
|  [**FTryllModelName**](struct_f_tryll_model_name.md) | [**TtsModelName**](#variable-ttsmodelname)  <br> |
|  FString | [**VoiceDescription**](#variable-voicedescription)  <br> |
|  bool | [**bEmitVisemes**](#variable-bemitvisemes)   = `false`<br> |


## Public Static Attributes

| Type | Name |
| ---: | :--- |
|  const TCHAR \* | [**NeutralMode**](#variable-neutralmode)  <br> |














## Public Functions

| Type | Name |
| ---: | :--- |
|  TArray&lt; FString &gt; | [**GetModeNames**](#function-getmodenames) () const<br> |
|  bool | [**HasMode**](#function-hasmode) (const FString & Name) const<br> |
|  void | [**ToWire**](#function-towire) (::Tryll::VoiceProfileT & Out) const<br> |
|  void | [**Validate**](#function-validate) (const FString & Context, TArray&lt; [**FTryllWorkflowIssue**](struct_f_tryll_workflow_issue.md) &gt; & OutIssues, bool bCheckFilesExist=true) const<br> |
|  void | [**ValidateAgainst**](#function-validateagainst) (const [**FTryllTtsCapabilities**](struct_f_tryll_tts_capabilities.md) & Caps, const FString & Context, TArray&lt; [**FTryllWorkflowIssue**](struct_f_tryll_workflow_issue.md) &gt; & OutIssues) const<br> |


## Public Static Functions

| Type | Name |
| ---: | :--- |
|  bool | [**IsValidModeName**](#function-isvalidmodename) (const FString & Name) <br> |


























## Detailed Description


The voice of one agent (protocol v27): the TTS model, the base identity, the speech-pipeline settings and the named delivery modes. Every speaking node of the agent (Speak, Generate And Speak) uses it. Hand-mirrors VoiceProfile in voice.fbs; ToWire converts it for CreateAgent / SetAgentVoice. 


    
## Public Attributes Documentation




### variable DefaultDeliveryMode 

```C++
FString FTryllVoiceProfileData::DefaultDeliveryMode;
```



Mode active after the voice is set. Empty = "neutral". 


        

<hr>



### variable DeliveryModes 

```C++
TArray<FTryllDeliveryMode> FTryllVoiceProfileData::DeliveryModes;
```



Named delivery modes. Game code switches between them with Set Delivery Mode. 


        

<hr>



### variable Expressiveness 

```C++
float FTryllVoiceProfileData::Expressiveness;
```



Chatterbox exaggeration used by "neutral" and by modes that do not set their own, 0.0-2.0. Ignored by other families. 


        

<hr>



### variable Lang 

```C++
FString FTryllVoiceProfileData::Lang;
```



Language for multilingual models that pick it per call. Empty = session locale, then the model default. Rejected on models with a fixed language. 


        

<hr>



### variable Prepare 

```C++
ETryllVoicePreparation FTryllVoiceProfileData::Prepare;
```



When the server prepares the reference clips. 


        

<hr>



### variable ReferenceVoice 

```C++
FTryllStoragePath FTryllVoiceProfileData::ReferenceVoice;
```



Clone families (Pocket TTS, Chatterbox, Qwen3-TTS Base): reference clip relative to the storage root; locale-aware (voices/loc/&lt;locale&gt;/). Empty = the model's default voice (Qwen3-TTS Base requires a clip). 


        

<hr>



### variable SpeakerId 

```C++
int32 FTryllVoiceProfileData::SpeakerId;
```



Speaker or style index for multi-speaker models (Supertonic 3: 0-9; Orpheus and Maya1: preset index). Must be below the model's speaker count. 


        

<hr>



### variable Speed 

```C++
float FTryllVoiceProfileData::Speed;
```



Speech-rate multiplier, 1.0 = native. Must stay 1.0 on models without speed control (Orpheus, Maya1, Qwen3-TTS). 


        

<hr>



### variable TtsModelName 

```C++
FTryllModelName FTryllVoiceProfileData::TtsModelName;
```



TTS model catalog name. Required: there is no default model. 


        

<hr>



### variable VoiceDescription 

```C++
FString FTryllVoiceProfileData::VoiceDescription;
```



Maya1: natural-language identity of the voice. Overrides Speaker Id. 


        

<hr>



### variable bEmitVisemes 

```C++
bool FTryllVoiceProfileData::bEmitVisemes;
```



Emit a viseme track for every sentence this agent speaks, from any speaking node. Turn on for lip-synced characters (Tryll Lip Sync, the MetaHuman Viseme solver). 


        

<hr>
## Public Static Attributes Documentation




### variable NeutralMode 

```C++
const TCHAR* FTryllVoiceProfileData::NeutralMode;
```



The built-in mode that is always available. 


        

<hr>
## Public Functions Documentation




### function GetModeNames 

```C++
TArray< FString > FTryllVoiceProfileData::GetModeNames () const
```



Names of the defined delivery modes. 


        

<hr>



### function HasMode 

```C++
bool FTryllVoiceProfileData::HasMode (
    const FString & Name
) const
```



True when Name is "neutral" or one of the defined modes. 


        

<hr>



### function ToWire 

```C++
void FTryllVoiceProfileData::ToWire (
    ::Tryll::VoiceProfileT & Out
) const
```



Fill the FlatBuffers object-API mirror. Storage paths are normalized like node file pickers. 


        

<hr>



### function Validate 

```C++
void FTryllVoiceProfileData::Validate (
    const FString & Context,
    TArray< FTryllWorkflowIssue > & OutIssues,
    bool bCheckFilesExist=true
) const
```



Local checks the server would also make, minus anything that needs the model's capabilities: a model name, mode names, the default mode, ranges, text limits and the reference clips (path shape and, when bCheckFilesExist and the storage folder is configured, that the file exists). Context names the profile in each message. 


        

<hr>



### function ValidateAgainst 

```C++
void FTryllVoiceProfileData::ValidateAgainst (
    const FTryllTtsCapabilities & Caps,
    const FString & Context,
    TArray< FTryllWorkflowIssue > & OutIssues
) const
```



The capability half of the server's check (InvalidVoiceProfile, 3023): a field the model cannot use is an error. Caps usually come from the catalog (ListModels or the editor's catalog file); the server checks the loaded model's runtime capabilities. 


        

<hr>
## Public Static Functions Documentation




### function IsValidModeName 

```C++
static bool FTryllVoiceProfileData::IsValidModeName (
    const FString & Name
) 
```



True for a valid delivery-mode name: [a-z0-9\_]{1,32}. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllVoiceProfile.h`

