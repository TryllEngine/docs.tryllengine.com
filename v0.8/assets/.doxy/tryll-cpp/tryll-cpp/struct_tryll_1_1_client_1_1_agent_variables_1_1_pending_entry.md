

# Struct Tryll::Client::AgentVariables::PendingEntry



[**ClassList**](annotated.md) **>** [**PendingEntry**](struct_tryll_1_1_client_1_1_agent_variables_1_1_pending_entry.md)



[More...](#detailed-description)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::vector&lt; std::string &gt; | [**adds**](#variable-adds)  <br> |
|  std::vector&lt; std::string &gt; | [**removes**](#variable-removes)  <br> |
|  bool | [**wholesale**](#variable-wholesale)   = `false`<br> |












































## Detailed Description


Per-name coalesced staging entry (see class doc). When `wholesale` is true the net effect is "assign the current mirror value" (a scalar set, a whole-set SetStringSet, or a Reset, possibly followed by set edits); `adds`/`removes` are ignored. Otherwise the entry is a pure set delta against the server's current set (sorted-unique, mutually exclusive). 


    
## Public Attributes Documentation




### variable adds 

```C++
std::vector<std::string> Tryll::Client::AgentVariables::PendingEntry::adds;
```




<hr>



### variable removes 

```C++
std::vector<std::string> Tryll::Client::AgentVariables::PendingEntry::removes;
```




<hr>



### variable wholesale 

```C++
bool Tryll::Client::AgentVariables::PendingEntry::wholesale;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/cpp/include/tryll/AgentVariables.h`

