

# Struct FTryllModelInfo



[**ClassList**](annotated.md) **>** [**FTryllModelInfo**](struct_f_tryll_model_info.md)



[More...](#detailed-description)

* `#include <TryllModelInfo.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Audience**](#variable-audience)  <br> |
|  int32 | [**ContextSize**](#variable-contextsize)   = `0`<br> |
|  [**FTryllSamplingParams**](struct_f_tryll_sampling_params.md) | [**DefaultSampling**](#variable-defaultsampling)  <br> |
|  TArray&lt; [**FTryllModelDependency**](struct_f_tryll_model_dependency.md) &gt; | [**Dependencies**](#variable-dependencies)  <br> |
|  FString | [**Engine**](#variable-engine)  <br> |
|  TArray&lt; FString &gt; | [**Files**](#variable-files)  <br> |
|  FString | [**HuggingFaceRepo**](#variable-huggingfacerepo)  <br> |
|  TArray&lt; FString &gt; | [**Languages**](#variable-languages)  <br> |
|  FString | [**LocalPath**](#variable-localpath)  <br> |
|  FString | [**ModelType**](#variable-modeltype)  <br> |
|  FString | [**Name**](#variable-name)  <br> |
|  int64 | [**SizeBytes**](#variable-sizebytes)   = `0`<br> |
|  ETryllModelStatus | [**Status**](#variable-status)   = `ETryllModelStatus::Absent`<br> |
|  FString | [**ToolCallFormat**](#variable-toolcallformat)  <br> |
|  FString | [**ToolCallSupport**](#variable-toolcallsupport)  <br> |
|  [**FTryllTtsCapabilities**](struct_f_tryll_tts_capabilities.md) | [**TtsCapabilities**](#variable-ttscapabilities)  <br> |
|  bool | [**bHasDefaultSampling**](#variable-bhasdefaultsampling)   = `false`<br> |
|  bool | [**bHasTtsCapabilities**](#variable-bhasttscapabilities)   = `false`<br> |












































## Detailed Description


Summary information for one model returned by ListModels. 


    
## Public Attributes Documentation




### variable Audience 

```C++
FString FTryllModelInfo::Audience;
```



Intended audience tier: "public" \| "experimental" \| "internal" \| "". 


        

<hr>



### variable ContextSize 

```C++
int32 FTryllModelInfo::ContextSize;
```



Variant context window in tokens; 0 means the engine default. 


        

<hr>



### variable DefaultSampling 

```C++
FTryllSamplingParams FTryllModelInfo::DefaultSampling;
```



Recommended sampling parameters; valid only when bHasDefaultSampling. 


        

<hr>



### variable Dependencies 

```C++
TArray<FTryllModelDependency> FTryllModelInfo::Dependencies;
```



Shared-asset dependencies of this model. Empty when the variant has none. 


        

<hr>



### variable Engine 

```C++
FString FTryllModelInfo::Engine;
```



Resolved inference engine, e.g. "llama-cpp" \| "sherpa-onnx". 


        

<hr>



### variable Files 

```C++
TArray<FString> FTryllModelInfo::Files;
```



Files downloaded/resolved for the configured engine's variant. 


        

<hr>



### variable HuggingFaceRepo 

```C++
FString FTryllModelInfo::HuggingFaceRepo;
```



HuggingFace repo id (e.g. "Qwen/Qwen2.5-0.5B-Instruct-GGUF") or empty for local-only models. 


        

<hr>



### variable Languages 

```C++
TArray<FString> FTryllModelInfo::Languages;
```



Languages this model can handle, as lowercase BCP-47 primary language subtags ("en", "de", ...). Read this to build a language menu instead of hardcoding one - it is how you discover that voice cloning is English-only, or which STT models cover the locale you want.


Empty means _undeclared_, not "supports nothing": treat it as "no
capability information" and keep offering the model. 


        

<hr>



### variable LocalPath 

```C++
FString FTryllModelInfo::LocalPath;
```



Resolved on-disk directory for the model; empty when not on disk. 


        

<hr>



### variable ModelType 

```C++
FString FTryllModelInfo::ModelType;
```



Model type: "language" \| "embedding" \| "stt" \| "tts" \| "vad". 


        

<hr>



### variable Name 

```C++
FString FTryllModelInfo::Name;
```



Catalog name of the model (matches the name in models.json). 


        

<hr>



### variable SizeBytes 

```C++
int64 FTryllModelInfo::SizeBytes;
```



On-disk size in bytes when the model has been downloaded. Zero if Status == Absent. 


        

<hr>



### variable Status 

```C++
ETryllModelStatus FTryllModelInfo::Status;
```



Current status on the server (absent / local / downloading / downloaded / loaded). 


        

<hr>



### variable ToolCallFormat 

```C++
FString FTryllModelInfo::ToolCallFormat;
```



Tool-call prompt/parse format: "chatml" \| "llama3" \| "mistral" \| "generic" \| "". 


        

<hr>



### variable ToolCallSupport 

```C++
FString FTryllModelInfo::ToolCallSupport;
```



Advisory catalog-declared tool-calling capability for language models: "supported" \| "unsupported". Empty for non-language models (and when talking to older servers). The authoritative check happens server-side at agent creation from the model's own chat template. 


        

<hr>



### variable TtsCapabilities 

```C++
FTryllTtsCapabilities FTryllModelInfo::TtsCapabilities;
```



Catalog TTS capabilities; valid only when bHasTtsCapabilities. 


        

<hr>



### variable bHasDefaultSampling 

```C++
bool FTryllModelInfo::bHasDefaultSampling;
```



True when the catalog supplies explicit default sampling parameters. 


        

<hr>



### variable bHasTtsCapabilities 

```C++
bool FTryllModelInfo::bHasTtsCapabilities;
```



True when the catalog advertised structured TTS capabilities. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllModelInfo.h`

