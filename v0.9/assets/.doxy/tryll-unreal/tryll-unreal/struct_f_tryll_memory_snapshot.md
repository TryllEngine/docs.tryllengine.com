

# Struct FTryllMemorySnapshot



[**ClassList**](annotated.md) **>** [**FTryllMemorySnapshot**](struct_f_tryll_memory_snapshot.md)



[More...](#detailed-description)

* `#include <TryllMemorySnapshot.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  int64 | [**CapturedAtMs**](#variable-capturedatms)   = `0`<br> |
|  TArray&lt; [**FTryllMemoryComponent**](struct_f_tryll_memory_component.md) &gt; | [**Components**](#variable-components)  <br> |
|  TArray&lt; [**FTryllMemoryDevice**](struct_f_tryll_memory_device.md) &gt; | [**Devices**](#variable-devices)  <br> |
|  int64 | [**ProcessPrivateBytes**](#variable-processprivatebytes)   = `0`<br> |
|  int64 | [**ProcessWorkingSetBytes**](#variable-processworkingsetbytes)   = `0`<br> |
|  int64 | [**UnattributedRamBytes**](#variable-unattributedrambytes)   = `0`<br> |
|  bool | [**bBackendInitialized**](#variable-bbackendinitialized)   = `false`<br> |
|  bool | [**bProcessAvailable**](#variable-bprocessavailable)   = `false`<br> |


















## Public Static Functions

| Type | Name |
| ---: | :--- |
|  [**FTryllMemorySnapshot**](struct_f_tryll_memory_snapshot.md) | [**FromWire**](#function-fromwire-12) (const ::Tryll::GetMemoryConsumptionResponse \* Resp) <br> |
|  [**FTryllMemorySnapshot**](struct_f_tryll_memory_snapshot.md) | [**FromWire**](#function-fromwire-22) (const ::Tryll::GetMemoryConsumptionResponseT & Resp) <br> |


























## Detailed Description


A complete memory picture at one instant, pulled over the wire. 


    
## Public Attributes Documentation




### variable CapturedAtMs 

```C++
int64 FTryllMemorySnapshot::CapturedAtMs;
```




<hr>



### variable Components 

```C++
TArray<FTryllMemoryComponent> FTryllMemorySnapshot::Components;
```




<hr>



### variable Devices 

```C++
TArray<FTryllMemoryDevice> FTryllMemorySnapshot::Devices;
```




<hr>



### variable ProcessPrivateBytes 

```C++
int64 FTryllMemorySnapshot::ProcessPrivateBytes;
```




<hr>



### variable ProcessWorkingSetBytes 

```C++
int64 FTryllMemorySnapshot::ProcessWorkingSetBytes;
```




<hr>



### variable UnattributedRamBytes 

```C++
int64 FTryllMemorySnapshot::UnattributedRamBytes;
```




<hr>



### variable bBackendInitialized 

```C++
bool FTryllMemorySnapshot::bBackendInitialized;
```




<hr>



### variable bProcessAvailable 

```C++
bool FTryllMemorySnapshot::bProcessAvailable;
```




<hr>
## Public Static Functions Documentation




### function FromWire [1/2]

```C++
static FTryllMemorySnapshot FTryllMemorySnapshot::FromWire (
    const ::Tryll::GetMemoryConsumptionResponse * Resp
) 
```




<hr>



### function FromWire [2/2]

```C++
static FTryllMemorySnapshot FTryllMemorySnapshot::FromWire (
    const ::Tryll::GetMemoryConsumptionResponseT & Resp
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllMemorySnapshot.h`

