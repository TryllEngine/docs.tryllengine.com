

# Struct FTryllModelName



[**ClassList**](annotated.md) **>** [**FTryllModelName**](struct_f_tryll_model_name.md)



[More...](#detailed-description)

* `#include <TryllModelName.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Name**](#variable-name)  <br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**FTryllModelName**](#function-ftryllmodelname-13) () = default<br> |
|   | [**FTryllModelName**](#function-ftryllmodelname-23) (const FString & InName) <br> |
|   | [**FTryllModelName**](#function-ftryllmodelname-33) (const TCHAR \* InName) <br> |
|  bool | [**IsEmpty**](#function-isempty) () const<br> |
|  const FString & | [**ToString**](#function-tostring) () const<br> |
|  bool | [**operator!=**](#function-operator) (const [**FTryllModelName**](struct_f_tryll_model_name.md) & Other) const<br> |
|  const TCHAR \* | [**operator\***](#function-operator_1) () const<br> |
|  bool | [**operator==**](#function-operator_2) (const [**FTryllModelName**](struct_f_tryll_model_name.md) & Other) const<br> |




























## Detailed Description


A model name chosen from the project's registered models (the TryllModelManifest).


In the editor this renders as a type-filtered dropdown plus the model's registration tier (Production / Experimental). Tag the property with meta=(TryllModelType="stt"\|"tts"\|"language"\|"embedding"\|"vad") to filter the list.


Implicitly converts to/from FString so it is mostly drop-in where a model-name string was used before. 


    
## Public Attributes Documentation




### variable Name 

```C++
FString FTryllModelName::Name;
```




<hr>
## Public Functions Documentation




### function FTryllModelName [1/3]

```C++
FTryllModelName::FTryllModelName () = default
```




<hr>



### function FTryllModelName [2/3]

```C++
inline FTryllModelName::FTryllModelName (
    const FString & InName
) 
```




<hr>



### function FTryllModelName [3/3]

```C++
inline FTryllModelName::FTryllModelName (
    const TCHAR * InName
) 
```




<hr>



### function IsEmpty 

```C++
inline bool FTryllModelName::IsEmpty () const
```




<hr>



### function ToString 

```C++
inline const FString & FTryllModelName::ToString () const
```




<hr>



### function operator!= 

```C++
inline bool FTryllModelName::operator!= (
    const FTryllModelName & Other
) const
```




<hr>



### function operator\* 

```C++
inline const TCHAR * FTryllModelName::operator* () const
```




<hr>



### function operator== 

```C++
inline bool FTryllModelName::operator== (
    const FTryllModelName & Other
) const
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllModelName.h`

