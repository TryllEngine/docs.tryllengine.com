

# Struct FTryllMemoryDevice



[**ClassList**](annotated.md) **>** [**FTryllMemoryDevice**](struct_f_tryll_memory_device.md)



[More...](#detailed-description)

* `#include <TryllMemorySnapshot.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Description**](#variable-description)  <br> |
|  int64 | [**FreeBytes**](#variable-freebytes)   = `0`<br> |
|  int32 | [**Index**](#variable-index)   = `-1`<br> |
|  FString | [**Kind**](#variable-kind)  <br> |
|  FString | [**Name**](#variable-name)  <br> |
|  int64 | [**OtherBytes**](#variable-otherbytes)   = `0`<br> |
|  int64 | [**SelfComputeBytes**](#variable-selfcomputebytes)   = `0`<br> |
|  int64 | [**SelfContextBytes**](#variable-selfcontextbytes)   = `0`<br> |
|  int64 | [**SelfModelBytes**](#variable-selfmodelbytes)   = `0`<br> |
|  int64 | [**TotalBytes**](#variable-totalbytes)   = `0`<br> |
















## Public Functions

| Type | Name |
| ---: | :--- |
|  int64 | [**GetSelfBytes**](#function-getselfbytes) () const<br> |




























## Detailed Description


One device's memory situation. 


    
## Public Attributes Documentation




### variable Description 

```C++
FString FTryllMemoryDevice::Description;
```




<hr>



### variable FreeBytes 

```C++
int64 FTryllMemoryDevice::FreeBytes;
```




<hr>



### variable Index 

```C++
int32 FTryllMemoryDevice::Index;
```



GPU index 0 is valid; host RAM is -1. 


        

<hr>



### variable Kind 

```C++
FString FTryllMemoryDevice::Kind;
```




<hr>



### variable Name 

```C++
FString FTryllMemoryDevice::Name;
```




<hr>



### variable OtherBytes 

```C++
int64 FTryllMemoryDevice::OtherBytes;
```




<hr>



### variable SelfComputeBytes 

```C++
int64 FTryllMemoryDevice::SelfComputeBytes;
```




<hr>



### variable SelfContextBytes 

```C++
int64 FTryllMemoryDevice::SelfContextBytes;
```




<hr>



### variable SelfModelBytes 

```C++
int64 FTryllMemoryDevice::SelfModelBytes;
```




<hr>



### variable TotalBytes 

```C++
int64 FTryllMemoryDevice::TotalBytes;
```




<hr>
## Public Functions Documentation




### function GetSelfBytes 

```C++
inline int64 FTryllMemoryDevice::GetSelfBytes () const
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllMemorySnapshot.h`

