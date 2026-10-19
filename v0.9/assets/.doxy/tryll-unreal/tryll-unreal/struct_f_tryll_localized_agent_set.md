

# Struct FTryllLocalizedAgentSet



[**ClassList**](annotated.md) **>** [**FTryllLocalizedAgentSet**](struct_f_tryll_localized_agent_set.md)



[More...](#detailed-description)

* `#include <TryllTextResolve.h>`

















## Public Types

| Type | Name |
| ---: | :--- |
| typedef TFunction&lt; void(uint64 AgentId, const FString &AssetName)&gt; | [**FApplied**](#typedef-fapplied)  <br> |
| typedef TFunction&lt; void(uint64 AgentId, const FString &AssetName, const [**FTryllError**](struct_f_tryll_error.md) &Error)&gt; | [**FFail**](#typedef-ffail)  <br> |
| typedef TFunction&lt; void( const TSharedPtr&lt; [**FTryllAgent**](class_f_tryll_agent.md) &gt; &Agent, const FString &NodeName, [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) \*Params, TFunction&lt; void(const [**FTryllError**](struct_f_tryll_error.md) &)&gt; OnComplete)&gt; | [**FSend**](#typedef-fsend)  <br> |




















## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**FTryllLocalizedAgentSet**](#function-ftrylllocalizedagentset-12) () <br> |
|   | [**FTryllLocalizedAgentSet**](#function-ftrylllocalizedagentset-22) (const [**FTryllLocalizedAgentSet**](struct_f_tryll_localized_agent_set.md) &) = delete<br> |
|  void | [**Forget**](#function-forget) (uint64 AgentId) <br> |
|  void | [**ForgetAll**](#function-forgetall) () <br> |
|  bool | [**HasPending**](#function-haspending) (uint64 AgentId) const<br> |
|  bool | [**IsTracked**](#function-istracked) (uint64 AgentId) const<br> |
|  void | [**MarkDirty**](#function-markdirty) (uint64 AgentId) <br> |
|  int32 | [**NoteTurnEnded**](#function-noteturnended) (uint64 AgentId, [**UTryllTextProvider**](class_u_tryll_text_provider.md) \* Provider, const FSend & Send, const FFail & Fail, const FApplied & Applied=FApplied()) <br> |
|  int32 | [**Num**](#function-num) () const<br> |
|  int32 | [**Refresh**](#function-refresh) ([**UTryllTextProvider**](class_u_tryll_text_provider.md) \* Provider, const FSend & Send, const FFail & Fail, const FApplied & Applied=FApplied()) <br> |
|  int32 | [**RefreshAgent**](#function-refreshagent) (uint64 AgentId, [**UTryllTextProvider**](class_u_tryll_text_provider.md) \* Provider, const FSend & Send, const FFail & Fail, const FApplied & Applied=FApplied()) <br> |
|  void | [**Track**](#function-track) (const TSharedPtr&lt; [**FTryllAgent**](class_f_tryll_agent.md) &gt; & Agent, FName Table, const FString & AssetName) <br> |
|  [**FTryllLocalizedAgentSet**](struct_f_tryll_localized_agent_set.md) & | [**operator=**](#function-operator) (const [**FTryllLocalizedAgentSet**](struct_f_tryll_localized_agent_set.md) &) = delete<br> |




























## Detailed Description


Live agents created with ticked Text IDs. On a language change each agent is refreshed from its current params baseline (so runtime ChangeParams survive): ticked non-structural fields are re-resolved on a clone and only nodes whose text changed are sent. One failed field sends nothing for that agent. An AgentBusy reply marks the agent dirty; the refresh is redone when its turn ends. Applied fires once the latest refresh of an agent is fully acknowledged, or at once when no text changed. A refresh started later makes the replies of an earlier one stale. 


    
## Public Types Documentation




### typedef FApplied 

```C++
using FTryllLocalizedAgentSet::FApplied = TFunction<void(uint64 AgentId, const FString& AssetName)>;
```




<hr>



### typedef FFail 

```C++
using FTryllLocalizedAgentSet::FFail = TFunction<void(uint64 AgentId, const FString& AssetName, const FTryllError& Error)>;
```




<hr>



### typedef FSend 

```C++
using FTryllLocalizedAgentSet::FSend = TFunction<void(
       const TSharedPtr<FTryllAgent>& Agent,
       const FString& NodeName,
       UTryllNodeParamsBase* Params,
       TFunction<void(const FTryllError&)> OnComplete)>;
```




<hr>
## Public Functions Documentation




### function FTryllLocalizedAgentSet [1/2]

```C++
FTryllLocalizedAgentSet::FTryllLocalizedAgentSet () 
```




<hr>



### function FTryllLocalizedAgentSet [2/2]

```C++
FTryllLocalizedAgentSet::FTryllLocalizedAgentSet (
    const FTryllLocalizedAgentSet &
) = delete
```




<hr>



### function Forget 

```C++
void FTryllLocalizedAgentSet::Forget (
    uint64 AgentId
) 
```




<hr>



### function ForgetAll 

```C++
void FTryllLocalizedAgentSet::ForgetAll () 
```




<hr>



### function HasPending 

```C++
bool FTryllLocalizedAgentSet::HasPending (
    uint64 AgentId
) const
```




<hr>



### function IsTracked 

```C++
bool FTryllLocalizedAgentSet::IsTracked (
    uint64 AgentId
) const
```




<hr>



### function MarkDirty 

```C++
void FTryllLocalizedAgentSet::MarkDirty (
    uint64 AgentId
) 
```




<hr>



### function NoteTurnEnded 

```C++
int32 FTryllLocalizedAgentSet::NoteTurnEnded (
    uint64 AgentId,
    UTryllTextProvider * Provider,
    const FSend & Send,
    const FFail & Fail,
    const FApplied & Applied=FApplied()
) 
```



The agent's turn ended: redo the refresh when an earlier send came back AgentBusy. 


        

<hr>



### function Num 

```C++
inline int32 FTryllLocalizedAgentSet::Num () const
```




<hr>



### function Refresh 

```C++
int32 FTryllLocalizedAgentSet::Refresh (
    UTryllTextProvider * Provider,
    const FSend & Send,
    const FFail & Fail,
    const FApplied & Applied=FApplied()
) 
```



Refresh every tracked agent. Returns the number of node sends. 


        

<hr>



### function RefreshAgent 

```C++
int32 FTryllLocalizedAgentSet::RefreshAgent (
    uint64 AgentId,
    UTryllTextProvider * Provider,
    const FSend & Send,
    const FFail & Fail,
    const FApplied & Applied=FApplied()
) 
```



Refresh one agent from its current baseline. Returns the number of node sends. 


        

<hr>



### function Track 

```C++
void FTryllLocalizedAgentSet::Track (
    const TSharedPtr< FTryllAgent > & Agent,
    FName Table,
    const FString & AssetName
) 
```




<hr>



### function operator= 

```C++
FTryllLocalizedAgentSet & FTryllLocalizedAgentSet::operator= (
    const FTryllLocalizedAgentSet &
) = delete
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllTextResolve.h`

