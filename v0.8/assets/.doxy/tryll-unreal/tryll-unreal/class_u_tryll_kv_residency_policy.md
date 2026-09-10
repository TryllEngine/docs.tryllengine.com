

# Class UTryllKvResidencyPolicy



[**ClassList**](annotated.md) **>** [**UTryllKvResidencyPolicy**](class_u_tryll_kv_residency_policy.md)



[More...](#detailed-description)

* `#include <TryllKvResidencyPolicy.h>`



Inherits the following classes: UActorComponent


















## Public Attributes

| Type | Name |
| ---: | :--- |
|  float | [**BusyRetrySeconds**](#variable-busyretryseconds)   = `0.5f`<br> |
|  int32 | [**HotSetSize**](#variable-hotsetsize)   = `2`<br> |
|  float | [**PrefillDebounceSeconds**](#variable-prefilldebounceseconds)   = `0.35f`<br> |
|  bool | [**bEnabled**](#variable-benabled)   = `true`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  [**UTryllAgentComponent**](class_u_tryll_agent_component.md) \* | [**GetActiveAgent**](#function-getactiveagent) () const<br> |
|  void | [**Register**](#function-register) ([**UTryllAgentComponent**](class_u_tryll_agent_component.md) \* Agent, bool bStartsResident) <br> |
|  void | [**RegisterAll**](#function-registerall) (bool bStartResident) <br> |
|  void | [**RestoreAll**](#function-restoreall) () <br> |
|  void | [**SetActiveAgent**](#function-setactiveagent) ([**UTryllAgentComponent**](class_u_tryll_agent_component.md) \* Agent) <br> |
| virtual void | [**TickComponent**](#function-tickcomponent) (float DeltaTime, ELevelTick TickType, FActorComponentTickFunction \* ThisTickFunction) override<br> |
|   | [**UTryllKvResidencyPolicy**](#function-utryllkvresidencypolicy) () <br> |
|  void | [**Unregister**](#function-unregister) ([**UTryllAgentComponent**](class_u_tryll_agent_component.md) \* Agent) <br> |




























## Detailed Description


Keeps only the agents currently in use holding language-model KV cache, evicting the rest to reclaim VRAM.


**This is a memory-for-latency trade, and it should be opt-in.** With every agent's contexts resident, switching between agents is nearly free. Evicted, the first message to an agent also pays context allocation and a full prefill — on the order of a second or two — in exchange for a large VRAM saving. That is the right trade on a GPU that would otherwise not fit and the wrong one on a GPU with headroom, which is why bEnabled exists rather than this being unconditional. Drive it from a graphics-quality or "low VRAM" setting.


**Why this is in the plugin rather than in your game.** Nothing here is game-specific, and every part of it is a mistake waiting to be made: forgetting the debounce turns scrubbing through a character list into one uncancellable prefill per entry; forgetting the hot set punishes the most common interaction pattern there is; and forgetting that cache operations are idle-only produces intermittent AgentBusy failures that look like server bugs.


**Correctness never depends on this.** The server transparently restores an evicted context before the next send, so a badly tuned policy costs time to first token, not behaviour.


**Usage.** Add to any Actor, call Register for each agent that should participate (or RegisterAll), and call SetActiveAgent whenever the player's focus moves. Agents that must always stay warm — a background summariser that runs after every turn, say — should simply not be registered: evicting one guarantees a restore on its next turn, which is strictly worse than never evicting it.


For creation-time behaviour, pair this with [**UTryllAgentComponent::KvCacheInitialization**](class_u_tryll_agent_component.md#variable-kvcacheinitialization) = DeferAllocation on the agents it manages. That is where most of the saving comes from: a context costs its full context\_size the moment it is created, so never allocating it is worth more than evicting it later. 


    
## Public Attributes Documentation




### variable BusyRetrySeconds 

```C++
float UTryllKvResidencyPolicy::BusyRetrySeconds;
```



Seconds to back off after an operation was refused with AgentBusy. 


        

<hr>



### variable HotSetSize 

```C++
int32 UTryllKvResidencyPolicy::HotSetSize;
```



How many agents keep their contexts resident, including the active one.


Two is the useful default: "go back to the one I was just using" is the commonest pattern, and a hot set of one punishes it with a full restore every time. One maximises the saving. 


        

<hr>



### variable PrefillDebounceSeconds 

```C++
float UTryllKvResidencyPolicy::PrefillDebounceSeconds;
```



Seconds an agent must stay active before its prefill starts.


Not cosmetic: a prefill cannot be cancelled once started, so the only way to avoid paying for agents the player merely passed over is not to start. 


        

<hr>



### variable bEnabled 

```C++
bool UTryllKvResidencyPolicy::bEnabled;
```



Whether the policy evicts at all. When false nothing is evicted, so toggling at runtime is safe; call RestoreAll afterwards to warm what was evicted.


Agents created with DeferAllocation stay that way — creation mode is structural. Turning the policy off mid-session restores what it evicted; it cannot undo an allocation that never happened. 


        

<hr>
## Public Functions Documentation




### function GetActiveAgent 

```C++
UTryllAgentComponent * UTryllKvResidencyPolicy::GetActiveAgent () const
```



The agent most recently passed to SetActiveAgent, or null. 


        

<hr>



### function Register 

```C++
void UTryllKvResidencyPolicy::Register (
    UTryllAgentComponent * Agent,
    bool bStartsResident
) 
```



Registers an agent. bStartsResident should be false when the agent was created with DeferAllocation (nothing is allocated yet) and true otherwise. Re-registering is a no-op. 


        

<hr>



### function RegisterAll 

```C++
void UTryllKvResidencyPolicy::RegisterAll (
    bool bStartResident
) 
```



Registers every agent component in the world. Convenience for simple setups. 


        

<hr>



### function RestoreAll 

```C++
void UTryllKvResidencyPolicy::RestoreAll () 
```



Restores every registered agent. Call after setting bEnabled to false so nobody is left paying restore costs for a policy that is no longer running. 


        

<hr>



### function SetActiveAgent 

```C++
void UTryllKvResidencyPolicy::SetActiveAgent (
    UTryllAgentComponent * Agent
) 
```



Tells the policy which agent the player is now interacting with. Safe to call on every selection, including repeats — the debounce and the resident set absorb the noise.


Named SetActiveAgent rather than SetActive because UActorComponent already has a SetActive, and a same-named override would both hide the base call and be rejected by UHT. 


        

<hr>



### function TickComponent 

```C++
virtual void UTryllKvResidencyPolicy::TickComponent (
    float DeltaTime,
    ELevelTick TickType,
    FActorComponentTickFunction * ThisTickFunction
) override
```




<hr>



### function UTryllKvResidencyPolicy 

```C++
UTryllKvResidencyPolicy::UTryllKvResidencyPolicy () 
```




<hr>



### function Unregister 

```C++
void UTryllKvResidencyPolicy::Unregister (
    UTryllAgentComponent * Agent
) 
```



Stops managing an agent. Does not evict or restore it. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllKvResidencyPolicy.h`

