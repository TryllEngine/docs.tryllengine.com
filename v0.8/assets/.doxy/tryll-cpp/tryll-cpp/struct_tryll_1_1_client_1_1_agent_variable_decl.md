

# Struct Tryll::Client::AgentVariableDecl



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**AgentVariableDecl**](struct_tryll_1_1_client_1_1_agent_variable_decl.md)



[More...](#detailed-description)

* `#include <AgentVariables.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  bool | [**allowOutputSubstitution**](#variable-allowoutputsubstitution)   = `false`<br> |
|  [**VariableValue**](namespace_tryll_1_1_client.md#typedef-variablevalue) | [**initial**](#variable-initial)  <br> |
|  std::string | [**name**](#variable-name)  <br> |












































## Detailed Description


One declared variable + its initial value, supplied to [**Tryll::Client::TryllClient::CreateAgent**](class_tryll_1_1_client_1_1_tryll_client.md#function-createagent-12). The complete declaration for an agent — every variable it will ever have — must be supplied there; post-create writes to an undeclared name fail locally (no wire call). 


    
## Public Attributes Documentation




### variable allowOutputSubstitution 

```C++
bool Tryll::Client::AgentVariableDecl::allowOutputSubstitution;
```



When true, this variable may appear as `__NAME__` in Generate / GenerateAndSpeak output. Immutable declaration metadata (default false). 


        

<hr>



### variable initial 

```C++
VariableValue Tryll::Client::AgentVariableDecl::initial;
```




<hr>



### variable name 

```C++
std::string Tryll::Client::AgentVariableDecl::name;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/AgentVariables.h`

