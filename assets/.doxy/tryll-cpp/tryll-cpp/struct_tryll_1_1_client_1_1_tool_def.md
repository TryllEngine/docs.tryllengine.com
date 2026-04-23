

# Struct Tryll::Client::ToolDef



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**ToolDef**](struct_tryll_1_1_client_1_1_tool_def.md)



_Definition of a callable tool for a ToolCall node._ 

* `#include <GraphDescription.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::string | [**description**](#variable-description)  <br>_Human-readable description shown to the model._  |
|  std::string | [**name**](#variable-name)  <br>_Tool name as the model will emit it._  |
|  std::vector&lt; [**ToolParamDef**](struct_tryll_1_1_client_1_1_tool_param_def.md) &gt; | [**parameters**](#variable-parameters)  <br>_Ordered list of parameters; may be empty._  |












































## Public Attributes Documentation




### variable description 

_Human-readable description shown to the model._ 
```C++
std::string Tryll::Client::ToolDef::description;
```




<hr>



### variable name 

_Tool name as the model will emit it._ 
```C++
std::string Tryll::Client::ToolDef::name;
```




<hr>



### variable parameters 

_Ordered list of parameters; may be empty._ 
```C++
std::vector<ToolParamDef> Tryll::Client::ToolDef::parameters;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/tryll-mono/server/client-cpp/include/tryll/GraphDescription.h`

