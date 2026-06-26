

# Class UTryllCannedResponseParams



[**ClassList**](annotated.md) **>** [**UTryllCannedResponseParams**](class_u_tryll_canned_response_params.md)



[More...](#detailed-description)

* `#include <TryllCannedResponseParams.h>`



Inherits the following classes: [UTryllNodeParamsBase](class_u_tryll_node_params_base.md)






















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**DefaultExit**](#variable-defaultexit)  <br> |
|  TArray&lt; FString &gt; | [**InlineStrings**](#variable-inlinestrings)  <br> |
|  ETryllCannedResponseSelectionStrategy | [**SelectionStrategy**](#variable-selectionstrategy)   = `ETryllCannedResponseSelectionStrategy::RoundRobin`<br> |
|  [**FTryllStoragePath**](struct_f_tryll_storage_path.md) | [**StringStorage**](#variable-stringstorage)  <br> |
































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
| virtual flatbuffers::Offset&lt; void &gt; | [**Pack**](class_u_tryll_node_params_base.md#function-pack) (flatbuffers::FlatBufferBuilder & Fbb, ::Tryll::NodeParams::NodeParams & OutType) const<br> |
|  ETryllNodeType return | [**static\_cast**](class_u_tryll_node_params_base.md#function-static_cast) (0) <br> |






















































## Detailed Description


Canned-response node backed by session string storage. Resolution priority (factory): named string\_storage → inline\_strings → server-wide default (default-canned-responses.txt from server-config). 


    
## Public Attributes Documentation




### variable DefaultExit 

```C++
FString UTryllCannedResponseParams::DefaultExit;
```



Default exit target after emitting a canned response. Empty string = END. Graph exit "default" — target node name; empty = END. 


        

<hr>



### variable InlineStrings 

```C++
TArray<FString> UTryllCannedResponseParams::InlineStrings;
```



Inline response list. Structural because it materialises the underlying StringStorage at construction. 


        

<hr>



### variable SelectionStrategy 

```C++
ETryllCannedResponseSelectionStrategy UTryllCannedResponseParams::SelectionStrategy;
```



How the next canned response is chosen from the storage list. 


        

<hr>



### variable StringStorage 

```C++
FTryllStoragePath UTryllCannedResponseParams::StringStorage;
```



Named session string storage. Mutable — rebinds the underlying storage. 


        

<hr>
## Public Functions Documentation




### function GetNodeType 

```C++
inline virtual ETryllNodeType UTryllCannedResponseParams::GetNodeType () override const
```



Implements [*UTryllNodeParamsBase::GetNodeType*](class_u_tryll_node_params_base.md#function-getnodetype)


<hr>



### function Pack 

```C++
virtual flatbuffers::Offset< void > UTryllCannedResponseParams::Pack (
    flatbuffers::FlatBufferBuilder & Fbb,
    Tryll::NodeParams::NodeParams & OutType
) override const
```



Implements [*UTryllNodeParamsBase::Pack*](class_u_tryll_node_params_base.md#function-pack)


<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllCannedResponseParams.h`

