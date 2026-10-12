

# Struct UTryllSubsystem::FPendingCreate



[**ClassList**](annotated.md) **>** [**FPendingCreate**](struct_u_tryll_subsystem_1_1_f_pending_create.md)


























## Public Attributes

| Type | Name |
| ---: | :--- |
|  [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) | [**Graph**](#variable-graph)  <br> |
|  TFunction&lt; FTryllOnAgentCreated &gt; | [**OnComplete**](#variable-oncomplete)  <br> |
|  std::uint64\_t | [**RequestId**](#variable-requestid)   = `0`<br> |
|  TArray&lt; [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) &gt; | [**Variables**](#variable-variables)  <br> |
|  bool | [**bEnableDiagnostics**](#variable-benablediagnostics)   = `false`<br> |












































## Public Attributes Documentation




### variable Graph 

```C++
FTryllGraphDescription UTryllSubsystem::FPendingCreate::Graph;
```




<hr>



### variable OnComplete 

```C++
TFunction<FTryllOnAgentCreated> UTryllSubsystem::FPendingCreate::OnComplete;
```




<hr>



### variable RequestId 

```C++
std::uint64_t UTryllSubsystem::FPendingCreate::RequestId;
```




<hr>



### variable Variables 

```C++
TArray<FTryllVariableDecl> UTryllSubsystem::FPendingCreate::Variables;
```




<hr>



### variable bEnableDiagnostics 

```C++
bool UTryllSubsystem::FPendingCreate::bEnableDiagnostics;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllSubsystem.h`

