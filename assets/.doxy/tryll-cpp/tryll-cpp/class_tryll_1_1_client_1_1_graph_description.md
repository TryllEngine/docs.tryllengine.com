

# Class Tryll::Client::GraphDescription



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md)



_Fluent builder for a workflow graph description sent to the server._ [More...](#detailed-description)

* `#include <GraphDescription.h>`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**NodeDesc**](struct_tryll_1_1_client_1_1_graph_description_1_1_node_desc.md) <br>_Internal serialisable description of one node._  |
| struct | [**RouteDesc**](struct_tryll_1_1_client_1_1_graph_description_1_1_route_desc.md) <br>_Internal serialisable description of one wire between two nodes._  |


## Public Types

| Type | Name |
| ---: | :--- |
| typedef std::unordered\_map&lt; std::string, std::string &gt; | [**Params**](#typedef-params)  <br>_String → string map of per-node parameters sent on the wire._  |




















## Public Functions

| Type | Name |
| ---: | :--- |
|  [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) & | [**AddNode**](#function-addnode) (std::string name, [**NodeType**](namespace_tryll_1_1_client.md#enum-nodetype) type, [**Params**](class_tryll_1_1_client_1_1_graph_description.md#typedef-params) params={}) <br>_Append a node to the graph._  |
|  [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) & | [**AddToolCallNode**](#function-addtoolcallnode) (std::string name, std::vector&lt; [**ToolDef**](struct_tryll_1_1_client_1_1_tool_def.md) &gt; tools, [**Params**](class_tryll_1_1_client_1_1_graph_description.md#typedef-params) params={}) <br>_Append a_ `ToolCall` _node with structured tool definitions._ |
|  const std::string & | [**GetDefaultModelName**](#function-getdefaultmodelname) () noexcept const<br> |
|  const std::vector&lt; [**NodeDesc**](struct_tryll_1_1_client_1_1_graph_description_1_1_node_desc.md) &gt; & | [**GetNodes**](#function-getnodes) () noexcept const<br> |
|  const std::vector&lt; [**RouteDesc**](struct_tryll_1_1_client_1_1_graph_description_1_1_route_desc.md) &gt; & | [**GetRoutes**](#function-getroutes) () noexcept const<br> |
|  const std::string & | [**GetStartNode**](#function-getstartnode) () noexcept const<br> |
|  [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) & | [**SetDefaultModelName**](#function-setdefaultmodelname) (std::string name) <br>_Set the fallback model name for nodes that do not specify_ `model_name` _._ |
|  [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) & | [**SetStartNode**](#function-setstartnode) (std::string name) <br>_Nominate which node receives the turn when the agent runs._  |
|  [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) & | [**Wire**](#function-wire) (std::string source, std::string exitName, std::string target) <br>_Add a wire (exit route) from one node to another._  |




























## Detailed Description


Typical usage: 
```C++
auto graph = Tryll::Client::GraphDescription{}
    .AddNode("gen", Tryll::Client::NodeType::Generate, {{"prompt", "..."}})
    .SetStartNode("gen")
    .SetDefaultModelName("qwen2.5-0.5b-instruct");
auto agent = client.CreateAgent(graph);
```



The graph is serialised into `CreateAgentRequest` on [**Tryll::TryllClient::CreateAgent**](class_tryll_1_1_tryll_client.md#function-createagent); there is no server-side validation until that point. 


    
## Public Types Documentation




### typedef Params 

_String → string map of per-node parameters sent on the wire._ 
```C++
using Tryll::Client::GraphDescription::Params = std::unordered_map<std::string, std::string>;
```




<hr>
## Public Functions Documentation




### function AddNode 

_Append a node to the graph._ 
```C++
GraphDescription & Tryll::Client::GraphDescription::AddNode (
    std::string name,
    NodeType type,
    Params params={}
) 
```





**Parameters:**


* `name` Graph-unique node name; used by [**Wire**](class_tryll_1_1_client_1_1_graph_description.md#function-wire) and [**SetStartNode**](class_tryll_1_1_client_1_1_graph_description.md#function-setstartnode) to reference this node. 
* `type` Node kind; see [**NodeType**](namespace_tryll_1_1_client.md#enum-nodetype). For `NodeType::ToolCall` prefer [**AddToolCallNode**](class_tryll_1_1_client_1_1_graph_description.md#function-addtoolcallnode). 
* `params` Key/value node parameters. All values are sent as strings on the wire.



**Returns:**

`*this` for fluent chaining. 





        

<hr>



### function AddToolCallNode 

_Append a_ `ToolCall` _node with structured tool definitions._
```C++
GraphDescription & Tryll::Client::GraphDescription::AddToolCallNode (
    std::string name,
    std::vector< ToolDef > tools,
    Params params={}
) 
```



Convenience over [**AddNode**](class_tryll_1_1_client_1_1_graph_description.md#function-addnode) that accepts a list of [**ToolDef**](struct_tryll_1_1_client_1_1_tool_def.md) rather than encoding tools into string params.




**Parameters:**


* `name` Graph-unique node name. 
* `tools` Tool definitions available to the model at this node. 
* `params` Additional key/value node parameters.



**Returns:**

`*this` for fluent chaining. 





        

<hr>



### function GetDefaultModelName 

```C++
inline const std::string & Tryll::Client::GraphDescription::GetDefaultModelName () noexcept const
```





**Returns:**

Default model name, or empty if none has been set. 





        

<hr>



### function GetNodes 

```C++
inline const std::vector< NodeDesc > & Tryll::Client::GraphDescription::GetNodes () noexcept const
```





**Returns:**

Nodes in insertion order. 





        

<hr>



### function GetRoutes 

```C++
inline const std::vector< RouteDesc > & Tryll::Client::GraphDescription::GetRoutes () noexcept const
```





**Returns:**

Wires in insertion order. 





        

<hr>



### function GetStartNode 

```C++
inline const std::string & Tryll::Client::GraphDescription::GetStartNode () noexcept const
```





**Returns:**

Start node name, or empty if none has been set. 





        

<hr>



### function SetDefaultModelName 

_Set the fallback model name for nodes that do not specify_ `model_name` _._
```C++
GraphDescription & Tryll::Client::GraphDescription::SetDefaultModelName (
    std::string name
) 
```



Sent as `default_model_name` in `CreateAgentRequest`. Nodes that invoke a language model (`Generate`, `ToolCall`) use this value when their own `model` param is unset.




**Parameters:**


* `name` Catalog model name as registered in `models.json`.



**Returns:**

`*this` for fluent chaining. 





        

<hr>



### function SetStartNode 

_Nominate which node receives the turn when the agent runs._ 
```C++
GraphDescription & Tryll::Client::GraphDescription::SetStartNode (
    std::string name
) 
```





**Parameters:**


* `name` Name of a node previously added via [**AddNode**](class_tryll_1_1_client_1_1_graph_description.md#function-addnode) or [**AddToolCallNode**](class_tryll_1_1_client_1_1_graph_description.md#function-addtoolcallnode).



**Returns:**

`*this` for fluent chaining. 





        

<hr>



### function Wire 

_Add a wire (exit route) from one node to another._ 
```C++
GraphDescription & Tryll::Client::GraphDescription::Wire (
    std::string source,
    std::string exitName,
    std::string target
) 
```





**Parameters:**


* `source` Name of the node the wire leaves. 
* `exitName` Name of the exit route on `source` (exit names are node-type-specific; see the node reference). 
* `target` Name of the node the wire enters.



**Returns:**

`*this` for fluent chaining. 





        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo/server/client-cpp/include/tryll/GraphDescription.h`

