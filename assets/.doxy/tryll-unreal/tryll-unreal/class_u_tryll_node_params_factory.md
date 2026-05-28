

# Class UTryllNodeParamsFactory



[**ClassList**](annotated.md) **>** [**UTryllNodeParamsFactory**](class_u_tryll_node_params_factory.md)



[More...](#detailed-description)

* `#include <TryllNodeParamsFactory.h>`



Inherits the following classes: UBlueprintFunctionLibrary


















## Public Attributes

| Type | Name |
| ---: | :--- |
|  UObject \* | [**WorldContextObject**](#variable-worldcontextobject)  <br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**UFUNCTION**](#function-ufunction-110) (BlueprintCallable, Category="Tryll\|Params", meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")) <br> |
|   | [**UFUNCTION**](#function-ufunction-210) (BlueprintCallable, Category="Tryll\|Params", meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")) <br> |
|   | [**UFUNCTION**](#function-ufunction-310) (BlueprintCallable, Category="Tryll\|Params", meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")) <br> |
|   | [**UFUNCTION**](#function-ufunction-410) (BlueprintCallable, Category="Tryll\|Params", meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")) <br> |
|   | [**UFUNCTION**](#function-ufunction-510) (BlueprintCallable, Category="Tryll\|Params", meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")) <br> |
|   | [**UFUNCTION**](#function-ufunction-610) (BlueprintCallable, Category="Tryll\|Params", meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")) <br> |
|   | [**UFUNCTION**](#function-ufunction-710) (BlueprintCallable, Category="Tryll\|Params", meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")) <br> |
|   | [**UFUNCTION**](#function-ufunction-810) (BlueprintCallable, Category="Tryll\|Params", meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")) <br> |
|   | [**UFUNCTION**](#function-ufunction-910) (BlueprintCallable, Category="Tryll\|Params", meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")) <br> |
|   | [**UFUNCTION**](#function-ufunction-1010) (BlueprintCallable, Category="Tryll\|Params", meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")) <br> |




























## Detailed Description


Blueprint-callable factory for constructing concrete node params objects.


Allows Blueprints to create typed params for ChangeParams() and graph authoring without any per-project C++ glue.


Usage (Blueprint): MakeGenerateParams → set fields → Agent.ChangeParams(NodeName, Params) CloneParams(Baseline) → set one field → Agent.ChangeParams(NodeName, Cloned) 


    
## Public Attributes Documentation




### variable WorldContextObject 

```C++
UObject* UTryllNodeParamsFactory::WorldContextObject;
```




<hr>
## Public Functions Documentation




### function UFUNCTION [1/10]

```C++
UTryllNodeParamsFactory::UFUNCTION (
    BlueprintCallable,
    Category="Tryll|Params",
    meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")
) 
```



Construct an authored GenerateParams with schema-default field values. 


        

<hr>



### function UFUNCTION [2/10]

```C++
UTryllNodeParamsFactory::UFUNCTION (
    BlueprintCallable,
    Category="Tryll|Params",
    meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")
) 
```



Construct an authored HumanMessageGuardrailParams with schema-default field values. 


        

<hr>



### function UFUNCTION [3/10]

```C++
UTryllNodeParamsFactory::UFUNCTION (
    BlueprintCallable,
    Category="Tryll|Params",
    meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")
) 
```



Construct an authored CannedResponseParams with schema-default field values. 


        

<hr>



### function UFUNCTION [4/10]

```C++
UTryllNodeParamsFactory::UFUNCTION (
    BlueprintCallable,
    Category="Tryll|Params",
    meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")
) 
```



Construct an authored ToolCallParams with schema-default field values. 


        

<hr>



### function UFUNCTION [5/10]

```C++
UTryllNodeParamsFactory::UFUNCTION (
    BlueprintCallable,
    Category="Tryll|Params",
    meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")
) 
```



Construct an authored RetrieveParams with schema-default field values. 


        

<hr>



### function UFUNCTION [6/10]

```C++
UTryllNodeParamsFactory::UFUNCTION (
    BlueprintCallable,
    Category="Tryll|Params",
    meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")
) 
```



Construct an authored InstructionParams with schema-default field values. 


        

<hr>



### function UFUNCTION [7/10]

```C++
UTryllNodeParamsFactory::UFUNCTION (
    BlueprintCallable,
    Category="Tryll|Params",
    meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")
) 
```



Construct an authored ClassifyIntentParams with schema-default field values. 


        

<hr>



### function UFUNCTION [8/10]

```C++
UTryllNodeParamsFactory::UFUNCTION (
    BlueprintCallable,
    Category="Tryll|Params",
    meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")
) 
```



Construct an authored ClassifyIntentLLMParams with schema-default field values. 


        

<hr>



### function UFUNCTION [9/10]

```C++
UTryllNodeParamsFactory::UFUNCTION (
    BlueprintCallable,
    Category="Tryll|Params",
    meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")
) 
```



Construct an authored IntentToInstructionParams with schema-default field values. 


        

<hr>



### function UFUNCTION [10/10]

```C++
UTryllNodeParamsFactory::UFUNCTION (
    BlueprintCallable,
    Category="Tryll|Params",
    meta=(WorldContext="WorldContextObject", DefaultToSelf="WorldContextObject")
) 
```



Deep-clone any [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) subclass. Typical mutation idiom: Clone = CloneParams(Agent-&gt;GetNodeParamsBaseline(NodeName)) // set fields on Clone Agent-&gt;ChangeParams(NodeName, Clone) 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/Generated/TryllNodeParamsFactory.h`

