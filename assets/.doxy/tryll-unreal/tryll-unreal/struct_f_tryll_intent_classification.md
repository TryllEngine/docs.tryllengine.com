

# Struct FTryllIntentClassification



[**ClassList**](annotated.md) **>** [**FTryllIntentClassification**](struct_f_tryll_intent_classification.md)



[More...](#detailed-description)

* `#include <TryllIntentClassification.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  int64 | [**AgentId**](#variable-agentid)   = `0`<br> |
|  float | [**Distance**](#variable-distance)   = `0.f`<br> |
|  FString | [**Intent**](#variable-intent)  <br> |
|  FString | [**NodeName**](#variable-nodename)  <br> |
|  FString | [**NotFoundReason**](#variable-notfoundreason)  <br> |
|  FString | [**RecordId**](#variable-recordid)  <br> |
|  int64 | [**RecordIndex**](#variable-recordindex)   = `0`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  bool | [**Accepted**](#function-accepted) () const<br> |




























## Detailed Description


Embedding ClassifyIntent payload (event\_type="intent\_classified"). 


    
## Public Attributes Documentation




### variable AgentId 

```C++
int64 FTryllIntentClassification::AgentId;
```




<hr>



### variable Distance 

```C++
float FTryllIntentClassification::Distance;
```




<hr>



### variable Intent 

```C++
FString FTryllIntentClassification::Intent;
```




<hr>



### variable NodeName 

```C++
FString FTryllIntentClassification::NodeName;
```




<hr>



### variable NotFoundReason 

```C++
FString FTryllIntentClassification::NotFoundReason;
```




<hr>



### variable RecordId 

```C++
FString FTryllIntentClassification::RecordId;
```




<hr>



### variable RecordIndex 

```C++
int64 FTryllIntentClassification::RecordIndex;
```




<hr>
## Public Functions Documentation




### function Accepted 

```C++
inline bool FTryllIntentClassification::Accepted () const
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllIntentClassification.h`

