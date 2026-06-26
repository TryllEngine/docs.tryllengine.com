

# Struct Tryll::Client::GraphDescription::NodeDesc



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**GraphDescription**](class_tryll_1_1_client_1_1_graph_description.md) **>** [**NodeDesc**](struct_tryll_1_1_client_1_1_graph_description_1_1_node_desc.md)



_Internal serialisable description of one node._ 

* `#include <GraphDescription.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::string | [**name**](#variable-name)  <br>_Graph-unique node name._  |
|  [**NodeParamsVariant**](namespace_tryll_1_1_client.md#typedef-nodeparamsvariant) | [**params**](#variable-params)  <br>_Typed node parameters (one variant per node type)._  |












































## Public Attributes Documentation




### variable name 

_Graph-unique node name._ 
```C++
std::string Tryll::Client::GraphDescription::NodeDesc::name;
```




<hr>



### variable params 

_Typed node parameters (one variant per node type)._ 
```C++
NodeParamsVariant Tryll::Client::GraphDescription::NodeDesc::params;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/GraphDescription.h`

