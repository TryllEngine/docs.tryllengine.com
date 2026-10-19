

# Struct FTryllDialogSummary



[**ClassList**](annotated.md) **>** [**FTryllDialogSummary**](struct_f_tryll_dialog_summary.md)



[More...](#detailed-description)

* `#include <TryllDialogSnapshot.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  uint32 | [**CoveredThrough**](#variable-coveredthrough)   = `0`<br> |
|  FString | [**Producer**](#variable-producer)  <br> |
|  FString | [**Text**](#variable-text)  <br> |












































## Detailed Description


Rolling conversation summary written by a Summarize node.


CoveredThrough is the index of the last interaction folded into Text, inclusive. The server rejects an import whose value is at or past the snapshot's interaction count. 


    
## Public Attributes Documentation




### variable CoveredThrough 

```C++
uint32 FTryllDialogSummary::CoveredThrough;
```




<hr>



### variable Producer 

```C++
FString FTryllDialogSummary::Producer;
```




<hr>



### variable Text 

```C++
FString FTryllDialogSummary::Text;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllDialogSnapshot.h`

