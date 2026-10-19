

# Class UTryllPauseParams



[**ClassList**](annotated.md) **>** [**UTryllPauseParams**](class_u_tryll_pause_params.md)



[More...](#detailed-description)

* `#include <TryllPauseParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**DefaultExit**](#variable-defaultexit)  <br> |
|  int32 | [**PauseTimeoutMs**](#variable-pausetimeoutms)   = `0`<br> |
|  FString | [**TimeoutExit**](#variable-timeoutexit)  <br> |
































## Public Functions

| Type | Name |
| ---: | :--- |
| virtual ETryllNodeType | [**GetNodeType**](#function-getnodetype) () override const<br> |
| virtual flatbuffers::Offset&lt; void &gt; | [**Pack**](#function-pack) (flatbuffers::FlatBufferBuilder & Fbb, Tryll::NodeParams::NodeParams & OutType) override const<br> |


## Public Functions inherited from UTryllNodeParamsBase

See [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)

| Type | Name |
| ---: | :--- |
| virtual TArray&lt; FString &gt; | [**GetExitTargetOptions**](class_u_tryll_node_params_base.md#function-getexittargetoptions) () const<br> |
| virtual ETryllNodeType | [**GetNodeType**](class_u_tryll_node_params_base.md#function-getnodetype) () const<br> |
|  FString | [**GetOwnerNodeName**](class_u_tryll_node_params_base.md#function-getownernodename) () const<br> |
| virtual bool | [**IsTtsConsumer**](class_u_tryll_node_params_base.md#function-isttsconsumer) () const<br> |
| virtual flatbuffers::Offset&lt; void &gt; | [**Pack**](class_u_tryll_node_params_base.md#function-pack) (flatbuffers::FlatBufferBuilder & Fbb, ::Tryll::NodeParams::NodeParams & OutType) const<br> |
| virtual void | [**PostLoad**](class_u_tryll_node_params_base.md#function-postload) () override<br> |
| virtual void | [**Serialize**](class_u_tryll_node_params_base.md#function-serialize) (FArchive & Ar) override<br> |
|  ETryllNodeType return | [**static\_cast**](class_u_tryll_node_params_base.md#function-static_cast) (0) <br> |




## Public Static Functions inherited from UTryllNodeParamsBase

See [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)

| Type | Name |
| ---: | :--- |
|  const [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) \* | [**FindEnclosingGraph**](class_u_tryll_node_params_base.md#function-findenclosinggraph) (const UObject \* Obj) <br> |












































## Protected Functions inherited from UTryllNodeParamsBase

See [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)

| Type | Name |
| ---: | :--- |
| virtual void | [**ApplyOldDefaults**](class_u_tryll_node_params_base.md#function-applyolddefaults) (int32 SavedVersion) <br> |
| virtual bool | [**UpgradeFrom**](class_u_tryll_node_params_base.md#function-upgradefrom) (int32 SavedVersion) <br> |






## Detailed Description


No-op node. The executor pauses the turn after this node exits, giving the client a chance to inspect/mutate agent state (via ChangeAgentParam) before the graph continues. Resume with ResumeAgentRequest. 


    
## Public Attributes Documentation




### variable DefaultExit 

```C++
FString UTryllPauseParams::DefaultExit;
```



Default exit target taken on a plain resume (empty resume\_node). Empty string = END. Graph exit "default" — target node name; empty = END. 


        

<hr>



### variable PauseTimeoutMs 

```C++
int32 UTryllPauseParams::PauseTimeoutMs;
```



Wall-clock budget for this pause, in milliseconds. 0 = infinite: wait for ResumeAgentRequest or CancelRequest however long that takes (the shipped default, so existing graphs are unaffected). A finite budget is the right choice whenever nothing guarantees the client will resume. Structural: resolved once at agent creation. 


        

<hr>



### variable TimeoutExit 

```C++
FString UTryllPauseParams::TimeoutExit;
```



Exit taken when pause\_timeout\_ms elapses. Empty = end the turn with TurnStatus.PauseTimedOut, keeping the partial interaction. Authoring a target lets the graph answer anyway (a canned fallback line, a Branch). Ignored when pause\_timeout\_ms is 0. Graph exit "timeout" — target node name; empty = END. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllPauseParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllPauseParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllPauseParams.h`

