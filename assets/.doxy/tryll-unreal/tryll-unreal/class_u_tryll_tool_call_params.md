

# Class UTryllToolCallParams



[**ClassList**](annotated.md) **>** [**UTryllToolCallParams**](class_u_tryll_tool_call_params.md)



[More...](#detailed-description)

* `#include <TryllToolCallParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  [**FTryllModelName**](struct_f_tryll_model_name.md) | [**ModelName**](#variable-modelname)  <br> |
|  FString | [**NoToolCalledExit**](#variable-notoolcalledexit)  <br> |
|  [**FTryllSamplingOverrides**](struct_f_tryll_sampling_overrides.md) | [**Sampling**](#variable-sampling)  <br> |
|  FString | [**SystemPrompt**](#variable-systemprompt)  <br> |
|  FString | [**ToolCallFormat**](#variable-toolcallformat)  <br> |
|  FString | [**ToolCalledExit**](#variable-toolcalledexit)  <br> |
|  TArray&lt; [**FTryllToolDefinition**](struct_f_tryll_tool_definition.md) &gt; | [**Tools**](#variable-tools)  <br> |
|  bool | [**bGenerateOnNoTool**](#variable-bgenerateonnotool)   = `false`<br> |
|  bool | [**bNotifyClient**](#variable-bnotifyclient)   = `false`<br> |
|  bool | [**bOverrideModelName**](#variable-boverridemodelname)   = `false`<br> |
|  bool | [**bOverrideSystemPrompt**](#variable-boverridesystemprompt)   = `false`<br> |
|  bool | [**bOverrideToolCallFormat**](#variable-boverridetoolcallformat)   = `false`<br> |
































## Public Functions

| Type | Name |
| ---: | :--- |
| virtual ETryllNodeType | [**GetNodeType**](#function-getnodetype) () override const<br> |
| virtual flatbuffers::Offset&lt; void &gt; | [**Pack**](#function-pack) (flatbuffers::FlatBufferBuilder & Fbb, Tryll::NodeParams::NodeParams & OutType) override const<br> |


## Public Functions inherited from UTryllNodeParamsBase

See [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)

| Type | Name |
| ---: | :--- |
| virtual TArray&lt; FString &gt; | [**GetExitTargetOptions**](class_u_tryll_node_params_base.md#function-getexittargetoptions) () const<br> |
| virtual ETryllNodeType | [**GetNodeType**](class_u_tryll_node_params_base.md#function-getnodetype) () const<br> |
| virtual flatbuffers::Offset&lt; void &gt; | [**Pack**](class_u_tryll_node_params_base.md#function-pack) (flatbuffers::FlatBufferBuilder & Fbb, ::Tryll::NodeParams::NodeParams & OutType) const<br> |
|  ETryllNodeType return | [**static\_cast**](class_u_tryll_node_params_base.md#function-static_cast) (0) <br> |






















































## Detailed Description


LLM-driven tool-call node. Constructs a tool-call prompt, runs sampling, parses the result. 


    
## Public Attributes Documentation




### variable ModelName 

```C++
FTryllModelName UTryllToolCallParams::ModelName;
```




<hr>



### variable NoToolCalledExit 

```C++
FString UTryllToolCallParams::NoToolCalledExit;
```



Exit taken when no tool call is parsed (or when generate\_on\_no\_tool produced a plain text reply). Empty string = END. Graph exit "no\_tool\_called" — target node name; empty = END. 


        

<hr>



### variable Sampling 

```C++
FTryllSamplingOverrides UTryllToolCallParams::Sampling;
```



Sparse sampling overrides. 


        

<hr>



### variable SystemPrompt 

```C++
FString UTryllToolCallParams::SystemPrompt;
```




<hr>



### variable ToolCallFormat 

```C++
FString UTryllToolCallParams::ToolCallFormat;
```




<hr>



### variable ToolCalledExit 

```C++
FString UTryllToolCallParams::ToolCalledExit;
```



Exit taken when one or more tool calls are parsed from the response. Empty string = END. Graph exit "tool\_called" — target node name; empty = END. 


        

<hr>



### variable Tools 

```C++
TArray<FTryllToolDefinition> UTryllToolCallParams::Tools;
```



Callable tool definitions. Structural because the tool-call prompt schema and parser contract are materialised at construction. 


        

<hr>



### variable bGenerateOnNoTool 

```C++
bool UTryllToolCallParams::bGenerateOnNoTool;
```



When no tool call is detected in the response, generate a plain text reply. 


        

<hr>



### variable bNotifyClient 

```C++
bool UTryllToolCallParams::bNotifyClient;
```



When true, emit an OnNodeEvent("tool\_call", …) for each tool call parsed. 


        

<hr>



### variable bOverrideModelName 

```C++
bool UTryllToolCallParams::bOverrideModelName;
```



Model catalog name. Empty = use the agent's default\_model\_name. Set to true to override the inherited ModelName value. 


        

<hr>



### variable bOverrideSystemPrompt 

```C++
bool UTryllToolCallParams::bOverrideSystemPrompt;
```



Prepended before the user turn during projection. Set to true to override the inherited SystemPrompt value. 


        

<hr>



### variable bOverrideToolCallFormat 

```C++
bool UTryllToolCallParams::bOverrideToolCallFormat;
```



Tool-call dialect used for both prompt construction and output parsing. Structural because the format config is baked into the projection strategy. Set to true to override the inherited ToolCallFormat value. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllToolCallParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllToolCallParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllToolCallParams.h`

