

# Class UTryllBranchParams



[**ClassList**](annotated.md) **>** [**UTryllBranchParams**](class_u_tryll_branch_params.md)



[More...](#detailed-description)

* `#include <TryllBranchParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Condition**](#variable-condition)  <br> |
|  FString | [**ElseExit**](#variable-elseexit)  <br> |
|  [**FTryllSlotInput**](struct_f_tryll_slot_input.md) | [**Input**](#variable-input)  <br> |
|  ETryllBranchTest | [**Test**](#variable-test)   = `ETryllBranchTest::NonEmpty`<br> |
|  FString | [**ThenExit**](#variable-thenexit)  <br> |
|  TArray&lt; FString &gt; | [**Values**](#variable-values)  <br> |
|  FString | [**Variable**](#variable-variable)  <br> |
|  bool | [**bNotifyClient**](#variable-bnotifyclient)   = `false`<br> |
|  bool | [**bOverrideCondition**](#variable-boverridecondition)   = `false`<br> |
|  bool | [**bOverrideInput**](#variable-boverrideinput)   = `false`<br> |
|  bool | [**bOverrideVariable**](#variable-boverridevariable)   = `false`<br> |
































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


Conditional routing node. Evaluates a test against a turn-local slot or an agent variable and takes then\_exit or else\_exit. Reader-only: no model, no slot write, no wire emission. 


    
## Public Attributes Documentation




### variable Condition 

```C++
FString UTryllBranchParams::Condition;
```




<hr>



### variable ElseExit 

```C++
FString UTryllBranchParams::ElseExit;
```



Exit taken when the test does not match, Expression is false, or the operand is missing. Empty = END. Graph exit "else" — target node name; empty = END. 


        

<hr>



### variable Input 

```C++
FTryllSlotInput UTryllBranchParams::Input;
```



Upstream inter-node slot this field consumes. Empty = context default. 


        

<hr>



### variable Test 

```C++
ETryllBranchTest UTryllBranchParams::Test;
```



Which predicate to run. Changing this recompiles the active condition. 


        

<hr>



### variable ThenExit 

```C++
FString UTryllBranchParams::ThenExit;
```



Exit taken when the test matches (or Expression is true). Empty = END. Graph exit "then" — target node name; empty = END. 


        

<hr>



### variable Values 

```C++
TArray<FString> UTryllBranchParams::Values;
```



Candidate literals for Equals / EqualsIgnoreCase / Contains / Regex. Match = any element matches. Unused under NonEmpty / IsTrue / Expression. Mutable — the game can remap cases between turns without recreating the agent. 


        

<hr>



### variable Variable 

```C++
FString UTryllBranchParams::Variable;
```




<hr>



### variable bNotifyClient 

```C++
bool UTryllBranchParams::bNotifyClient;
```



When true, fire OnNodeEvent("branch\_taken", …) after routing. 


        

<hr>



### variable bOverrideCondition 

```C++
bool UTryllBranchParams::bOverrideCondition;
```



JSON filter condition used when test=Expression. Supports the retrieve filter grammar plus `{"slot":"<name>"}` operands. Unused under other tests. Set to true to override the inherited Condition value. 


        

<hr>



### variable bOverrideInput 

```C++
bool UTryllBranchParams::bOverrideInput;
```



Slot discriminant. Empty = "user\_message". Mutually exclusive with a non-empty `variable`. Unused under test=Expression. Set to true to override the inherited Input value. 


        

<hr>



### variable bOverrideVariable 

```C++
bool UTryllBranchParams::bOverrideVariable;
```



Variable discriminant (name of a declared agent variable). Mutually exclusive with a non-empty `input`. Unused under test=Expression. Set to true to override the inherited Variable value. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllBranchParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllBranchParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllBranchParams.h`

