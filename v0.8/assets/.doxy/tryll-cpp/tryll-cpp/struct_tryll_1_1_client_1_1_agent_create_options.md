

# Struct Tryll::Client::AgentCreateOptions



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**AgentCreateOptions**](struct_tryll_1_1_client_1_1_agent_create_options.md)



[More...](#detailed-description)

* `#include <AgentCreateOptions.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  bool | [**enableDiagnostics**](#variable-enablediagnostics)   = `false`<br> |
|  ::Tryll::AgentKvCacheInitialization | [**kvCacheInitialization**](#variable-kvcacheinitialization)   = `/* multi line expression */`<br> |
|  bool | [**maintainDialogueHistory**](#variable-maintaindialoguehistory)   = `true`<br> |
|  std::vector&lt; [**AgentVariableDecl**](struct_tryll_1_1_client_1_1_agent_variable_decl.md) &gt; | [**variables**](#variable-variables)  <br> |
|  ::Tryll::AgentWorkload | [**workload**](#variable-workload)   = `::Tryll::AgentWorkload\_Interactive`<br> |












































## Detailed Description


Options for [**TryllClient::CreateAgent**](class_tryll_1_1_client_1_1_tryll_client.md#function-createagent-12) / CreateAgentAsync. Prefer this value object over positional booleans when adding new fields. 


    
## Public Attributes Documentation




### variable enableDiagnostics 

```C++
bool Tryll::Client::AgentCreateOptions::enableDiagnostics;
```




<hr>



### variable kvCacheInitialization 

```C++
::Tryll::AgentKvCacheInitialization Tryll::Client::AgentCreateOptions::kvCacheInitialization;
```




<hr>



### variable maintainDialogueHistory 

```C++
bool Tryll::Client::AgentCreateOptions::maintainDialogueHistory;
```




<hr>



### variable variables 

```C++
std::vector<AgentVariableDecl> Tryll::Client::AgentCreateOptions::variables;
```




<hr>



### variable workload 

```C++
::Tryll::AgentWorkload Tryll::Client::AgentCreateOptions::workload;
```



Whether a human ever waits on this agent's turns. Set Background for world-logic/evaluator agents so the inference scheduler paces or defers their work first under GPU pressure. Purely a scheduling hint. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/AgentCreateOptions.h`

