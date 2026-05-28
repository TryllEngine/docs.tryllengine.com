

# Struct FTryllWorkflowValidator



[**ClassList**](annotated.md) **>** [**FTryllWorkflowValidator**](struct_f_tryll_workflow_validator.md)












































## Public Static Functions

| Type | Name |
| ---: | :--- |
|  TArray&lt; [**FTryllWorkflowIssue**](struct_f_tryll_workflow_issue.md) &gt; | [**Validate**](#function-validate) (const [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) & Graph) <br> |
|  bool | [**ValidateAndLog**](#function-validateandlog) (const [**FTryllGraphDescription**](struct_f_tryll_graph_description.md) & Graph, const FString & ContextName) <br> |


























## Public Static Functions Documentation




### function Validate 

```C++
static TArray< FTryllWorkflowIssue > FTryllWorkflowValidator::Validate (
    const FTryllGraphDescription & Graph
) 
```



Validate `Graph` and return a (possibly empty) list of issues. Thread-safety: reads Graph fields only; safe to call from any thread. 


        

<hr>



### function ValidateAndLog 

```C++
static bool FTryllWorkflowValidator::ValidateAndLog (
    const FTryllGraphDescription & Graph,
    const FString & ContextName
) 
```



Convenience wrapper: validate and log results via UE\_LOG (LogTryllClient). Returns true if no errors were found (warnings are tolerated). 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/TryllWorkflowValidator.h`

