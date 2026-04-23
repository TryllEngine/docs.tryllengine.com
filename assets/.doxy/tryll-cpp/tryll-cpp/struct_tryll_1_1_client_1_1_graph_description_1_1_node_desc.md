

# Struct Tryll::Client::GraphDescription::NodeDesc



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) **>** [**NodeDesc**](struct_tryll_1_1_client_1_1_graph_description_1_1_node_desc.md)



_Internal serialisable description of one node._ 

* `#include <GraphDescription.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::string | [**name**](#variable-name)  <br>_Graph-unique node name._  |
|  [**Params**](class_tryll_1_1_client_1_1_graph_description.md#typedef-params) | [**params**](#variable-params)  <br>_Key/value node parameters (string-typed on the wire)._  |
|  std::vector&lt; [**ToolDef**](struct_tryll_1_1_client_1_1_tool_def.md) &gt; | [**tools**](#variable-tools)  <br>_Tool definitions; populated for_ `ToolCall` _nodes only._ |
|  [**NodeType**](namespace_tryll_1_1_client.md#enum-nodetype) | [**type**](#variable-type)  <br>_Node kind; see_ [_**NodeType**_](namespace_tryll_1_1_client.md#enum-nodetype) _._ |












































## Public Attributes Documentation




### variable name 

_Graph-unique node name._ 
```C++
std::string Tryll::Client::GraphDescription::NodeDesc::name;
```




<hr>



### variable params 

_Key/value node parameters (string-typed on the wire)._ 
```C++
Params Tryll::Client::GraphDescription::NodeDesc::params;
```




<hr>



### variable tools 

_Tool definitions; populated for_ `ToolCall` _nodes only._
```C++
std::vector<ToolDef> Tryll::Client::GraphDescription::NodeDesc::tools;
```




<hr>



### variable type 

_Node kind; see_ [_**NodeType**_](namespace_tryll_1_1_client.md#enum-nodetype) _._
```C++
NodeType Tryll::Client::GraphDescription::NodeDesc::type;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/tryll-mono/server/client-cpp/include/tryll/GraphDescription.h`

