

# Class Tryll::Client::GraphDescription



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md)



_Fluent builder for a workflow graph description sent to the server._ [More...](#detailed-description)

* `#include <GraphDescription.h>`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**NodeDesc**](struct_tryll_1_1_client_1_1_graph_description_1_1_node_desc.md) <br>_Internal serialisable description of one node._  |






















## Public Functions

| Type | Name |
| ---: | :--- |
|  [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) & | [**AddCannedResponse**](#function-addcannedresponse) (std::string name, ::Tryll::NodeParams::CannedResponseParamsT params) <br> |
|  [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) & | [**AddClassifyIntent**](#function-addclassifyintent) (std::string name, ::Tryll::NodeParams::ClassifyIntentParamsT params) <br> |
|  [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) & | [**AddClassifyIntentLLM**](#function-addclassifyintentllm) (std::string name, ::Tryll::NodeParams::ClassifyIntentLLMParamsT params) <br> |
|  [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) & | [**AddGenerate**](#function-addgenerate) (std::string name, ::Tryll::NodeParams::GenerateParamsT params) <br> |
|  [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) & | [**AddHumanMessageGuardrail**](#function-addhumanmessageguardrail) (std::string name, ::Tryll::NodeParams::HumanMessageGuardrailParamsT params) <br> |
|  [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) & | [**AddInstruction**](#function-addinstruction) (std::string name, ::Tryll::NodeParams::InstructionParamsT params) <br> |
|  [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) & | [**AddIntentToInstruction**](#function-addintenttoinstruction) (std::string name, ::Tryll::NodeParams::IntentToInstructionParamsT params) <br> |
|  [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) & | [**AddRetrieve**](#function-addretrieve) (std::string name, ::Tryll::NodeParams::RetrieveParamsT params) <br> |
|  [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) & | [**AddToolCall**](#function-addtoolcall) (std::string name, ::Tryll::NodeParams::ToolCallParamsT params) <br> |
|  const std::string & | [**GetDefaultModelName**](#function-getdefaultmodelname) () noexcept const<br> |
|  const std::vector&lt; [**NodeDesc**](struct_tryll_1_1_client_1_1_graph_description_1_1_node_desc.md) &gt; & | [**GetNodes**](#function-getnodes) () noexcept const<br> |
|  const std::string & | [**GetStartNode**](#function-getstartnode) () noexcept const<br> |
|  [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) & | [**SetDefaultModelName**](#function-setdefaultmodelname) (std::string name) <br>_Set the fallback model name for nodes that do not specify model\_name._  |
|  [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) & | [**SetStartNode**](#function-setstartnode) (std::string name) <br> |




























## Detailed Description


Typical usage: 
```C++
using namespace Tryll::NodeParams;
auto graph = Tryll::Client::GraphDescription{}
    .AddGenerate("gen", GenerateParamsT{.model_name="qwen2.5-0.5b-instruct",
                                       .system_prompt="You are helpful."})
    .SetStartNode("gen");
```
 


    
## Public Functions Documentation




### function AddCannedResponse 

```C++
GraphDescription & Tryll::Client::GraphDescription::AddCannedResponse (
    std::string name,
    ::Tryll::NodeParams::CannedResponseParamsT params
) 
```




<hr>



### function AddClassifyIntent 

```C++
GraphDescription & Tryll::Client::GraphDescription::AddClassifyIntent (
    std::string name,
    ::Tryll::NodeParams::ClassifyIntentParamsT params
) 
```




<hr>



### function AddClassifyIntentLLM 

```C++
GraphDescription & Tryll::Client::GraphDescription::AddClassifyIntentLLM (
    std::string name,
    ::Tryll::NodeParams::ClassifyIntentLLMParamsT params
) 
```




<hr>



### function AddGenerate 

```C++
GraphDescription & Tryll::Client::GraphDescription::AddGenerate (
    std::string name,
    ::Tryll::NodeParams::GenerateParamsT params
) 
```




<hr>



### function AddHumanMessageGuardrail 

```C++
GraphDescription & Tryll::Client::GraphDescription::AddHumanMessageGuardrail (
    std::string name,
    ::Tryll::NodeParams::HumanMessageGuardrailParamsT params
) 
```




<hr>



### function AddInstruction 

```C++
GraphDescription & Tryll::Client::GraphDescription::AddInstruction (
    std::string name,
    ::Tryll::NodeParams::InstructionParamsT params
) 
```




<hr>



### function AddIntentToInstruction 

```C++
GraphDescription & Tryll::Client::GraphDescription::AddIntentToInstruction (
    std::string name,
    ::Tryll::NodeParams::IntentToInstructionParamsT params
) 
```




<hr>



### function AddRetrieve 

```C++
GraphDescription & Tryll::Client::GraphDescription::AddRetrieve (
    std::string name,
    ::Tryll::NodeParams::RetrieveParamsT params
) 
```




<hr>



### function AddToolCall 

```C++
GraphDescription & Tryll::Client::GraphDescription::AddToolCall (
    std::string name,
    ::Tryll::NodeParams::ToolCallParamsT params
) 
```




<hr>



### function GetDefaultModelName 

```C++
inline const std::string & Tryll::Client::GraphDescription::GetDefaultModelName () noexcept const
```




<hr>



### function GetNodes 

```C++
inline const std::vector< NodeDesc > & Tryll::Client::GraphDescription::GetNodes () noexcept const
```




<hr>



### function GetStartNode 

```C++
inline const std::string & Tryll::Client::GraphDescription::GetStartNode () noexcept const
```




<hr>



### function SetDefaultModelName 

_Set the fallback model name for nodes that do not specify model\_name._ 
```C++
GraphDescription & Tryll::Client::GraphDescription::SetDefaultModelName (
    std::string name
) 
```




<hr>



### function SetStartNode 

```C++
GraphDescription & Tryll::Client::GraphDescription::SetStartNode (
    std::string name
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-cpp/include/tryll/GraphDescription.h`

