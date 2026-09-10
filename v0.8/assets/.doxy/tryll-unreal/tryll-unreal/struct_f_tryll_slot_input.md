

# Struct FTryllSlotInput



[**ClassList**](annotated.md) **>** [**FTryllSlotInput**](struct_f_tryll_slot_input.md)



[More...](#detailed-description)

* `#include <TryllSlotInput.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Value**](#variable-value)  <br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**FTryllSlotInput**](#function-ftryllslotinput-13) () = default<br> |
|   | [**FTryllSlotInput**](#function-ftryllslotinput-23) (const FString & InValue) <br> |
|   | [**FTryllSlotInput**](#function-ftryllslotinput-33) (const TCHAR \* InValue) <br> |
|  bool | [**IsEmpty**](#function-isempty) () const<br> |
|  const FString & | [**ToString**](#function-tostring) () const<br> |
|  bool | [**operator!=**](#function-operator) (const [**FTryllSlotInput**](struct_f_tryll_slot_input.md) & Other) const<br> |
|  const TCHAR \* | [**operator\***](#function-operator_1) () const<br> |
|  bool | [**operator==**](#function-operator_2) (const [**FTryllSlotInput**](struct_f_tryll_slot_input.md) & Other) const<br> |




























## Detailed Description


Selects an upstream inter-node slot ("input"-style fields: GenerateParams::Input, GenerateAndSpeakParams::Input, RetrieveParams::Input, SpeakParams::Input, ClassifyIntentParams::Input, ClassifyIntentLLMParams::Input, ToolCallParams::Input).


In the editor this renders as a two-kind dropdown — "User Message" plus one "Slot &lt;X&gt;" entry per slot reachable upstream of the owning node on some path — no free-text mode. Empty Value = "user\_message" (every input-bearing node, including Speak, shares this default).


Implicitly converts to/from FString so it is mostly drop-in where a slot-name string was used before. 


    
## Public Attributes Documentation




### variable Value 

```C++
FString FTryllSlotInput::Value;
```




<hr>
## Public Functions Documentation




### function FTryllSlotInput [1/3]

```C++
FTryllSlotInput::FTryllSlotInput () = default
```




<hr>



### function FTryllSlotInput [2/3]

```C++
inline FTryllSlotInput::FTryllSlotInput (
    const FString & InValue
) 
```




<hr>



### function FTryllSlotInput [3/3]

```C++
inline FTryllSlotInput::FTryllSlotInput (
    const TCHAR * InValue
) 
```




<hr>



### function IsEmpty 

```C++
inline bool FTryllSlotInput::IsEmpty () const
```




<hr>



### function ToString 

```C++
inline const FString & FTryllSlotInput::ToString () const
```




<hr>



### function operator!= 

```C++
inline bool FTryllSlotInput::operator!= (
    const FTryllSlotInput & Other
) const
```




<hr>



### function operator\* 

```C++
inline const TCHAR * FTryllSlotInput::operator* () const
```




<hr>



### function operator== 

```C++
inline bool FTryllSlotInput::operator== (
    const FTryllSlotInput & Other
) const
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllSlotInput.h`

