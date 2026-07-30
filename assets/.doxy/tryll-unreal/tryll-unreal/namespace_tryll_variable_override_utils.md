

# Namespace TryllVariableOverrideUtils



[**Namespace List**](namespaces.md) **>** [**TryllVariableOverrideUtils**](namespace_tryll_variable_override_utils.md)



[More...](#detailed-description)






































## Public Functions

| Type | Name |
| ---: | :--- |
|  [**FTryllError**](struct_f_tryll_error.md) | [**ApplyEffectiveValue**](#function-applyeffectivevalue) ([**FTryllAgentVariables**](class_f_tryll_agent_variables.md) & Vars, const [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) & Declared, const [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) \* OverrideValue, bool bEnabled) <br> |
|  [**FTryllError**](struct_f_tryll_error.md) | [**AssignFromDecl**](#function-assignfromdecl) ([**FTryllAgentVariables**](class_f_tryll_agent_variables.md) & Vars, const FString & Name, ETryllVariableType Type, const [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) & Source) <br> |
|  [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) | [**CloneDecl**](#function-clonedecl) (const [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) & Source) <br> |
|  TArray&lt; int32 &gt; | [**CollectLegacyIndices**](#function-collectlegacyindices) (const TArray&lt; [**FTryllVariableOverride**](struct_f_tryll_variable_override.md) &gt; & Overrides, const TArray&lt; [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) &gt; & Declared) <br> |
|  [**FTryllVariableOverride**](struct_f_tryll_variable_override.md) \* | [**EnsureEntry**](#function-ensureentry) (TArray&lt; [**FTryllVariableOverride**](struct_f_tryll_variable_override.md) &gt; & Overrides, const [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) & Decl) <br> |
|  int32 | [**FindFirstOverrideIndex**](#function-findfirstoverrideindex) (const TArray&lt; [**FTryllVariableOverride**](struct_f_tryll_variable_override.md) &gt; & Overrides, const FString & Name) <br> |
|  TArray&lt; [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) &gt; | [**MergeVariableOverrides**](#function-mergevariableoverrides) (const TArray&lt; [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) &gt; & Declared, const TArray&lt; [**FTryllVariableOverride**](struct_f_tryll_variable_override.md) &gt; & Overrides) <br> |
|  int32 | [**RemoveLegacyEntries**](#function-removelegacyentries) (TArray&lt; [**FTryllVariableOverride**](struct_f_tryll_variable_override.md) &gt; & Overrides, const TArray&lt; [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) &gt; & Declared) <br> |
|  TArray&lt; [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) &gt; | [**ResolveDeclared**](#function-resolvedeclared) (const [**UTryllAgentComponent**](class_u_tryll_agent_component.md) \* Component) <br> |
|  void | [**SeedFromDeclaration**](#function-seedfromdeclaration) ([**FTryllVariableOverride**](struct_f_tryll_variable_override.md) & Override, const [**FTryllVariableDecl**](struct_f_tryll_variable_decl.md) & Decl) <br> |




























## Detailed Description


Pure helpers for Material-Instance style variable overrides on [**UTryllAgentComponent**](class_u_tryll_agent_component.md). Used by CreateAgent merge, the Details customization, and automation specs — no Slate / property-handle dependency. 


    
## Public Functions Documentation




### function ApplyEffectiveValue 

```C++
FTryllError TryllVariableOverrideUtils::ApplyEffectiveValue (
    FTryllAgentVariables & Vars,
    const FTryllVariableDecl & Declared,
    const FTryllVariableDecl * OverrideValue,
    bool bEnabled
) 
```



Push the effective Inspector value onto the live agent mirror. When bEnabled is true, assign OverrideValue; when false, assign the workflow declaration default (not Reset(), which restores CreateAgent-time merged initials that may themselves have been overrides). 


        

<hr>



### function AssignFromDecl 

```C++
FTryllError TryllVariableOverrideUtils::AssignFromDecl (
    FTryllAgentVariables & Vars,
    const FString & Name,
    ETryllVariableType Type,
    const FTryllVariableDecl & Source
) 
```




<hr>



### function CloneDecl 

```C++
FTryllVariableDecl TryllVariableOverrideUtils::CloneDecl (
    const FTryllVariableDecl & Source
) 
```




<hr>



### function CollectLegacyIndices 

```C++
TArray< int32 > TryllVariableOverrideUtils::CollectLegacyIndices (
    const TArray< FTryllVariableOverride > & Overrides,
    const TArray< FTryllVariableDecl > & Declared
) 
```



Indices of overrides that should not appear as declaration-driven rows: empty names, names absent from the declaration set, or duplicate names after the first occurrence. Order is ascending index. 


        

<hr>



### function EnsureEntry 

```C++
FTryllVariableOverride * TryllVariableOverrideUtils::EnsureEntry (
    TArray< FTryllVariableOverride > & Overrides,
    const FTryllVariableDecl & Decl
) 
```



Ensure a sparse override entry exists for Decl. When newly created or its nested type disagrees with the declaration, seed name/type/value from Decl. Does not change bEnabled. 


        

<hr>



### function FindFirstOverrideIndex 

```C++
int32 TryllVariableOverrideUtils::FindFirstOverrideIndex (
    const TArray< FTryllVariableOverride > & Overrides,
    const FString & Name
) 
```



Index of the first override whose Name equals Name, or INDEX\_NONE. 


        

<hr>



### function MergeVariableOverrides 

```C++
TArray< FTryllVariableDecl > TryllVariableOverrideUtils::MergeVariableOverrides (
    const TArray< FTryllVariableDecl > & Declared,
    const TArray< FTryllVariableOverride > & Overrides
) 
```



Merge enabled matching overrides onto declarations (CreateAgent path). Type-mismatched and stale enabled overrides are skipped with LogTryll warnings. 


        

<hr>



### function RemoveLegacyEntries 

```C++
int32 TryllVariableOverrideUtils::RemoveLegacyEntries (
    TArray< FTryllVariableOverride > & Overrides,
    const TArray< FTryllVariableDecl > & Declared
) 
```



Remove legacy/stale entries (highest index first). Returns count removed. 


        

<hr>



### function ResolveDeclared 

```C++
TArray< FTryllVariableDecl > TryllVariableOverrideUtils::ResolveDeclared (
    const UTryllAgentComponent * Component
) 
```



Declarations that drive override UI: WorkflowAsset-&gt;Variables when set, else InlineVariables. 


        

<hr>



### function SeedFromDeclaration 

```C++
void TryllVariableOverrideUtils::SeedFromDeclaration (
    FTryllVariableOverride & Override,
    const FTryllVariableDecl & Decl
) 
```



Overwrite an override's nested value from the workflow declaration default. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllVariableOverrideUtils.h`

