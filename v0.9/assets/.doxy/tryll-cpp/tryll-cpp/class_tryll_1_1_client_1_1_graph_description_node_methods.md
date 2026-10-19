

# Class Tryll::Client::GraphDescriptionNodeMethods



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md)



[More...](#detailed-description)

* `#include <GraphDescription.Nodes.gen.h>`





































## Public Functions

| Type | Name |
| ---: | :--- |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddBranch**](#function-addbranch) (std::string name, ::Tryll::NodeParams::BranchParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddCannedResponse**](#function-addcannedresponse) (std::string name, ::Tryll::NodeParams::CannedResponseParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddClassifyAttributesLLM**](#function-addclassifyattributesllm) (std::string name, ::Tryll::NodeParams::ClassifyAttributesLLMParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddClassifyIntent**](#function-addclassifyintent) (std::string name, ::Tryll::NodeParams::ClassifyIntentParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddClassifyIntentLLM**](#function-addclassifyintentllm) (std::string name, ::Tryll::NodeParams::ClassifyIntentLLMParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddGenerate**](#function-addgenerate) (std::string name, ::Tryll::NodeParams::GenerateParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddGenerateAndSpeak**](#function-addgenerateandspeak) (std::string name, ::Tryll::NodeParams::GenerateAndSpeakParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddInstruction**](#function-addinstruction) (std::string name, ::Tryll::NodeParams::InstructionParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddIntentToInstruction**](#function-addintenttoinstruction) (std::string name, ::Tryll::NodeParams::IntentToInstructionParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddPause**](#function-addpause) (std::string name, ::Tryll::NodeParams::PauseParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddRegexGuardrail**](#function-addregexguardrail) (std::string name, ::Tryll::NodeParams::RegexGuardrailParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddRetrieve**](#function-addretrieve) (std::string name, ::Tryll::NodeParams::RetrieveParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddSpeak**](#function-addspeak) (std::string name, ::Tryll::NodeParams::SpeakParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddSummarize**](#function-addsummarize) (std::string name, ::Tryll::NodeParams::SummarizeParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddToolCall**](#function-addtoolcall) (std::string name, ::Tryll::NodeParams::ToolCallParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddTransform**](#function-addtransform) (std::string name, ::Tryll::NodeParams::TransformParamsT params) <br> |
|  [**GraphDescriptionNodeMethods**](class_tryll_1_1_client_1_1_graph_description_node_methods.md) & | [**AddWriteToMemory**](#function-addwritetomemory) (std::string name, ::Tryll::NodeParams::WriteToMemoryParamsT params) <br> |




























## Detailed Description


Typed AddX methods generated from the NodeParams union. Mixed into [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) via inheritance or friend-include. 


    
## Public Functions Documentation




### function AddBranch 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddBranch (
    std::string name,
    ::Tryll::NodeParams::BranchParamsT params
) 
```



Add a Branch node. Conditional routing node. Evaluates a test against a turn-local slot or an agent variable and takes then\_exit or else\_exit. Reader-only: no model, no slot write, no wire emission. 


        

<hr>



### function AddCannedResponse 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddCannedResponse (
    std::string name,
    ::Tryll::NodeParams::CannedResponseParamsT params
) 
```



Add a CannedResponse node. Canned-response node backed by session string storage. Resolution priority (factory): named string\_storage → inline\_strings → server-wide default (default-canned-responses.txt from server-config). 


        

<hr>



### function AddClassifyAttributesLLM 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddClassifyAttributesLLM (
    std::string name,
    ::Tryll::NodeParams::ClassifyAttributesLLMParamsT params
) 
```



Add a ClassifyAttributesLLM node. Several specialist classifiers scored in one cached batched call. Each attribute writes slot.&lt;output\_name&gt;\_&lt;name&gt;. triggered\_exit is taken when any attribute's own rule fires. 


        

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



Add a ClassifyIntentLLM node. Intent classification node (LLM first-token logprob over letter candidates). Scores labels via a dedicated LLM, attaches an IntentionComponent on found, and writes the decision to slot.&lt;output\_name&gt;. 


        

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



### function AddGenerateAndSpeak 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddGenerateAndSpeak (
    std::string name,
    ::Tryll::NodeParams::GenerateAndSpeakParamsT params
) 
```



Add a GenerateAndSpeak node. LLM-driven text generation + streaming TTS synthesis in a single fused node. Streams LLM tokens, segments them into sentences, synthesizes each sentence on a dedicated TTS strand, and emits PCM frames via OnTtsAudio callbacks. The voice (model, speaker, delivery mode, visemes) belongs to the agent: CreateAgentRequest.voice. Agent creation fails without one (3025). 


        

<hr>



### function AddInstruction 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddInstruction (
    std::string name,
    ::Tryll::NodeParams::InstructionParamsT params
) 
```



Add a Instruction node. Plain instruction node. Attaches a kind=Instruction SlotComponent to the current interaction so downstream Mustache templates can reference it. 


        

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



### function AddPause 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddPause (
    std::string name,
    ::Tryll::NodeParams::PauseParamsT params
) 
```



Add a Pause node. No-op node. The executor pauses the turn after this node exits, giving the client a chance to inspect/mutate agent state (via ChangeAgentParam) before the graph continues. Resume with ResumeAgentRequest. 


        

<hr>



### function AddRegexGuardrail 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddRegexGuardrail (
    std::string name,
    ::Tryll::NodeParams::RegexGuardrailParamsT params
) 
```



Add a RegexGuardrail node. Regex guardrail node backed by session string storage. Branches on whether the node's input slot text matches a pattern in the storage. Resolution priority (factory): named string\_storage → inline\_strings → server-wide default (default-guardrail-patterns.txt from server-config). 


        

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



### function AddSpeak 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddSpeak (
    std::string name,
    ::Tryll::NodeParams::SpeakParamsT params
) 
```



Add a Speak node. Standalone TTS-only node. Voices its `input`-resolved slot (any upstream node's output — Generate, CannedResponse, etc. — or `user_message` itself), segments it into sentences, and synthesizes each on a dedicated TTS strand, emitting PCM frames via OnTtsAudio callbacks. Unlike GenerateAndSpeak it runs no LLM inference, so it can voice canned/scripted lines or any upstream-produced answer without paying for a generation pass. The voice belongs to the agent (CreateAgentRequest.voice); agent creation fails without one (3025). 


        

<hr>



### function AddSummarize 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddSummarize (
    std::string name,
    ::Tryll::NodeParams::SummarizeParamsT params
) 
```



Add a Summarize node. Rolling conversation summary. Every time the history since the last fold crosses a trigger, this node compresses the older interactions (everything outside `keep_recent_turns`) into a SummaryComponent. Generate-class projection then renders that summary at the end of its system message and replays history only from `covered_through + 1`, so a long dialogue stays coherent at a fixed token cost instead of being trimmed away. Place it last in the graph, after the node that produced the player-facing reply. 


        

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



### function AddTransform 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddTransform (
    std::string name,
    ::Tryll::NodeParams::TransformParamsT params
) 
```



Add a Transform node. Model-free input-composition node. Renders a Mustache template over `user_message` and the current turn's slots into its own slot; performs no inference and never emits on the wire (`send = None`, `history_role = none`, fixed). Typical use: query rewriting, fan-out composition feeding a downstream Retrieve/Generate `input`. 


        

<hr>



### function AddWriteToMemory 

```C++
GraphDescriptionNodeMethods & Tryll::Client::GraphDescriptionNodeMethods::AddWriteToMemory (
    std::string name,
    ::Tryll::NodeParams::WriteToMemoryParamsT params
) 
```



Add a WriteToMemory node. Upsert the current input slot into a named mutable store. Empty input writes nothing and takes written\_exit; only real write failures take failed\_exit. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/cpp/include/tryll/Generated/GraphDescription.Nodes.gen.h`

