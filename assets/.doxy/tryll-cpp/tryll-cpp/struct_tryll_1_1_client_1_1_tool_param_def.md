

# Struct Tryll::Client::ToolParamDef



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**ToolParamDef**](struct_tryll_1_1_client_1_1_tool_param_def.md)



_Description of one parameter of a tool definition._ [More...](#detailed-description)

* `#include <GraphDescription.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::string | [**description**](#variable-description)  <br>_Human-readable description shown to the model._  |
|  std::string | [**name**](#variable-name)  <br>_Parameter name as the model will emit it._  |
|  std::string | [**type**](#variable-type)  <br>_Type hint, e.g. "string", "integer", "boolean"._  |












































## Detailed Description


Bundled into a [**ToolDef**](struct_tryll_1_1_client_1_1_tool_def.md) and shown to the model in the tool-calling prompt produced by a `ToolCall` node. 


    
## Public Attributes Documentation




### variable description 

_Human-readable description shown to the model._ 
```C++
std::string Tryll::Client::ToolParamDef::description;
```




<hr>



### variable name 

_Parameter name as the model will emit it._ 
```C++
std::string Tryll::Client::ToolParamDef::name;
```




<hr>



### variable type 

_Type hint, e.g. "string", "integer", "boolean"._ 
```C++
std::string Tryll::Client::ToolParamDef::type;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/tryll-mono/server/client-cpp/include/tryll/GraphDescription.h`

