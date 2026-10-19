

# Class UTryllWriteToMemoryParams



[**ClassList**](annotated.md) **>** [**UTryllWriteToMemoryParams**](class_u_tryll_write_to_memory_params.md)



[More...](#detailed-description)

* `#include <TryllWriteToMemoryParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**FailedExit**](#variable-failedexit)  <br> |
|  FString | [**Id**](#variable-id)  <br> |
|  [**FTryllSlotInput**](struct_f_tryll_slot_input.md) | [**Input**](#variable-input)  <br> |
|  FString | [**Schema**](#variable-schema)  <br> |
|  FString | [**StorageName**](#variable-storagename)  <br> |
|  FString | [**WrittenExit**](#variable-writtenexit)  <br> |
|  bool | [**bOverrideId**](#variable-boverrideid)   = `false`<br> |
|  bool | [**bOverrideInput**](#variable-boverrideinput)   = `false`<br> |
































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


Upsert the current input slot into a named mutable store. Empty input writes nothing and takes written\_exit; only real write failures take failed\_exit. 


    
## Public Attributes Documentation




### variable FailedExit 

```C++
FString UTryllWriteToMemoryParams::FailedExit;
```



Exit taken when the write is rejected (overflow, schema mismatch). Graph exit "failed" — target node name; empty = END. 


        

<hr>



### variable Id 

```C++
FString UTryllWriteToMemoryParams::Id;
```




<hr>



### variable Input 

```C++
FTryllSlotInput UTryllWriteToMemoryParams::Input;
```



Upstream inter-node slot this field consumes. Empty = context default. 


        

<hr>



### variable Schema 

```C++
FString UTryllWriteToMemoryParams::Schema;
```



Catalog schema name. Structural. 


        

<hr>



### variable StorageName 

```C++
FString UTryllWriteToMemoryParams::StorageName;
```



Session-scoped store name. May contain var.\* templates resolved at agent creation. Structural. 


        

<hr>



### variable WrittenExit 

```C++
FString UTryllWriteToMemoryParams::WrittenExit;
```



Exit taken after a successful write or a no-op empty input. Graph exit "written" — target node name; empty = END. 


        

<hr>



### variable bOverrideId 

```C++
bool UTryllWriteToMemoryParams::bOverrideId;
```



Record id, optionally templated with var.\* and resolved per turn. Empty asks the store to assign an insertion-ordered id. Set to true to override the inherited Id value. 


        

<hr>



### variable bOverrideInput 

```C++
bool UTryllWriteToMemoryParams::bOverrideInput;
```



Slot whose text is written. Empty = "user\_message". Set to true to override the inherited Input value. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllWriteToMemoryParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllWriteToMemoryParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllWriteToMemoryParams.h`

