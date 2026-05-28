

# Class Tryll::Client::GraphDescriptionNodeMethods



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md)



[More...](#detailed-description)

* `#include <GraphDescription.Nodes.gen.h>`





































## Public Functions

| Type | Name |
| ---: | :--- |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddCannedResponse**](#function-addcannedresponse) (std::string name, ::Tryll::NodeParams::CannedResponseParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddClassifyIntent**](#function-addclassifyintent) (std::string name, ::Tryll::NodeParams::ClassifyIntentParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddClassifyIntentLLM**](#function-addclassifyintentllm) (std::string name, ::Tryll::NodeParams::ClassifyIntentLLMParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddGenerate**](#function-addgenerate) (std::string name, ::Tryll::NodeParams::GenerateParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddHumanMessageGuardrail**](#function-addhumanmessageguardrail) (std::string name, ::Tryll::NodeParams::HumanMessageGuardrailParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddInstruction**](#function-addinstruction) (std::string name, ::Tryll::NodeParams::InstructionParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddIntentToInstruction**](#function-addintenttoinstruction) (std::string name, ::Tryll::NodeParams::IntentToInstructionParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddRetrieve**](#function-addretrieve) (std::string name, ::Tryll::NodeParams::RetrieveParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddToolCall**](#function-addtoolcall) (std::string name, ::Tryll::NodeParams::ToolCallParamsT params) <br> |




























## Detailed Description


Typed AddX methods generated from the NodeParams union. Mixed into [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) via inheritance or friend-include. 


    
## Public Functions Documentation




### function AddCannedResponse 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddCannedResponse (
    std::string name,
    ::Tryll::NodeParams::CannedResponseParamsT params
) 
```



Add a CannedResponse node. Canned-response node backed by session string storage. Resolution priority (factory): named string\_storage → inline\_strings → server-wide default (default-canned-responses.txt from server-config). 


        

<hr>



### function AddClassifyIntent 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddClassifyIntent (
    std::string name,
    ::Tryll::NodeParams::ClassifyIntentParamsT params
) 
```



Add a ClassifyIntent node. Intent classification node (embedding-based, top-1 nearest-neighbour). Searches a labelled embedded storage and attaches an IntentionComponent. 


        

<hr>



### function AddClassifyIntentLLM 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddClassifyIntentLLM (
    std::string name,
    ::Tryll::NodeParams::ClassifyIntentLLMParamsT params
) 
```



Add a ClassifyIntentLLM node. Intent classification node (LLM first-token logprob over letter candidates). Scores labels via a dedicated LLM and attaches an IntentionComponent. 


        

<hr>



### function AddGenerate 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddGenerate (
    std::string name,
    ::Tryll::NodeParams::GenerateParamsT params
) 
```



Add a Generate node. LLM-driven text generation node. Renders a Mustache template, runs sampling, produces a streamed AnswerText. 


        

<hr>



### function AddHumanMessageGuardrail 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddHumanMessageGuardrail (
    std::string name,
    ::Tryll::NodeParams::HumanMessageGuardrailParamsT params
) 
```



Add a HumanMessageGuardrail node. Guardrail node backed by session string storage. Blocks turns whose human message matches a pattern in the storage. Resolution priority (factory): named string\_storage → inline\_strings → server-wide default (default-guardrail-patterns.txt from server-config). 


        

<hr>



### function AddInstruction 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddInstruction (
    std::string name,
    ::Tryll::NodeParams::InstructionParamsT params
) 
```



Add a Instruction node. Plain instruction node. Attaches an InstructionComponent to the current interaction so downstream Mustache templates can reference it. 


        

<hr>



### function AddIntentToInstruction 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddIntentToInstruction (
    std::string name,
    ::Tryll::NodeParams::IntentToInstructionParamsT params
) 
```



Add a IntentToInstruction node. Maps an intent label to an instruction text from a Map-kind string storage. Resolution priority (factory): named string\_storage → inline\_keys + inline\_strings (Map-kind in-memory storage built from the equal-length pair). 


        

<hr>



### function AddRetrieve 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddRetrieve (
    std::string name,
    ::Tryll::NodeParams::RetrieveParamsT params
) 
```



Add a Retrieve node. RAG retrieval node. Embeds the HumanMessage, searches an embedded storage, and attaches a KnowledgeComponent to the current interaction. 


        

<hr>



### function AddToolCall 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddToolCall (
    std::string name,
    ::Tryll::NodeParams::ToolCallParamsT params
) 
```



Add a ToolCall node. LLM-driven tool-call node. Constructs a tool-call prompt, runs sampling, parses the result. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-cpp/include/tryll/Generated/GraphDescription.Nodes.gen.h`

