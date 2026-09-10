

# Struct FTryllSlotName



[**ClassList**](annotated.md) **>** [**FTryllSlotName**](struct_f_tryll_slot_name.md)



[More...](#detailed-description)

* `#include <TryllSlotName.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Value**](#variable-value)  <br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**FTryllSlotName**](#function-ftryllslotname-13) () = default<br> |
|   | [**FTryllSlotName**](#function-ftryllslotname-23) (const FString & InValue) <br> |
|   | [**FTryllSlotName**](#function-ftryllslotname-33) (const TCHAR \* InValue) <br> |
|  bool | [**IsEmpty**](#function-isempty) () const<br> |
|  const FString & | [**ToString**](#function-tostring) () const<br> |
|  bool | [**operator!=**](#function-operator) (const [**FTryllSlotName**](struct_f_tryll_slot_name.md) & Other) const<br> |
|  const TCHAR \* | [**operator\***](#function-operator_1) () const<br> |
|  bool | [**operator==**](#function-operator_2) (const [**FTryllSlotName**](struct_f_tryll_slot_name.md) & Other) const<br> |




























## Detailed Description


Names a node's effective output slot ("output\_name" fields). Empty = the node's own name.


In the editor this renders a rename-commit widget: grammar ([A-Za-z\_][A-Za-z0-9\_]\*) + reserved-name ("user\_message") enforcement, uniquify against existing names/slots, and propagation of the old slot name into every `input` selector and `{{slot.<old>}}` template tag in the graph on commit.


Implicitly converts to/from FString so it is mostly drop-in where a slot-name string was used before. 


    
## Public Attributes Documentation




### variable Value 

```C++
FString FTryllSlotName::Value;
```




<hr>
## Public Functions Documentation




### function FTryllSlotName [1/3]

```C++
FTryllSlotName::FTryllSlotName () = default
```




<hr>



### function FTryllSlotName [2/3]

```C++
inline FTryllSlotName::FTryllSlotName (
    const FString & InValue
) 
```




<hr>



### function FTryllSlotName [3/3]

```C++
inline FTryllSlotName::FTryllSlotName (
    const TCHAR * InValue
) 
```




<hr>



### function IsEmpty 

```C++
inline bool FTryllSlotName::IsEmpty () const
```




<hr>



### function ToString 

```C++
inline const FString & FTryllSlotName::ToString () const
```




<hr>



### function operator!= 

```C++
inline bool FTryllSlotName::operator!= (
    const FTryllSlotName & Other
) const
```




<hr>



### function operator\* 

```C++
inline const TCHAR * FTryllSlotName::operator* () const
```




<hr>



### function operator== 

```C++
inline bool FTryllSlotName::operator== (
    const FTryllSlotName & Other
) const
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllSlotName.h`

