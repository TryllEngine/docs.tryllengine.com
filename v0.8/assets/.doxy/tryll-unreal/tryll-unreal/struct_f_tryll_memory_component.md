

# Struct FTryllMemoryComponent



[**ClassList**](annotated.md) **>** [**FTryllMemoryComponent**](struct_f_tryll_memory_component.md)



[More...](#detailed-description)

* `#include <TryllMemorySnapshot.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  int64 | [**AgentId**](#variable-agentid)   = `0`<br> |
|  int32 | [**CapacityTokens**](#variable-capacitytokens)   = `0`<br> |
|  int64 | [**ComputeBytes**](#variable-computebytes)   = `0`<br> |
|  ETryllInferenceConsumerClass | [**ConsumerClass**](#variable-consumerclass)   = `ETryllInferenceConsumerClass::Unspecified`<br> |
|  int32 | [**DeviceIndex**](#variable-deviceindex)   = `-1`<br> |
|  FString | [**EstimateBasis**](#variable-estimatebasis)  <br> |
|  FString | [**Kind**](#variable-kind)  <br> |
|  int64 | [**LastTurnAtMs**](#variable-lastturnatms)   = `0`<br> |
|  FString | [**Name**](#variable-name)  <br> |
|  FString | [**NodeName**](#variable-nodename)  <br> |
|  ETryllMemoryOwnerKind | [**Owner**](#variable-owner)   = `ETryllMemoryOwnerKind::Model`<br> |
|  TArray&lt; [**FTryllMemoryComponentDevice**](struct_f_tryll_memory_component_device.md) &gt; | [**PerDevice**](#variable-perdevice)  <br> |
|  int64 | [**RamBytes**](#variable-rambytes)   = `0`<br> |
|  int64 | [**RamBytesHigh**](#variable-rambyteshigh)   = `0`<br> |
|  int64 | [**SessionId**](#variable-sessionid)   = `0`<br> |
|  ETryllMemoryTier | [**Tier**](#variable-tier)   = `ETryllMemoryTier::Measured`<br> |
|  int32 | [**UsedTokens**](#variable-usedtokens)   = `0`<br> |
|  int64 | [**VramBytes**](#variable-vrambytes)   = `0`<br> |
|  bool | [**bOccupancyStale**](#variable-boccupancystale)   = `false`<br> |
|  bool | [**bResident**](#variable-bresident)   = `true`<br> |












































## Detailed Description


One row of the memory breakdown. 


    
## Public Attributes Documentation




### variable AgentId 

```C++
int64 FTryllMemoryComponent::AgentId;
```




<hr>



### variable CapacityTokens 

```C++
int32 FTryllMemoryComponent::CapacityTokens;
```




<hr>



### variable ComputeBytes 

```C++
int64 FTryllMemoryComponent::ComputeBytes;
```




<hr>



### variable ConsumerClass 

```C++
ETryllInferenceConsumerClass FTryllMemoryComponent::ConsumerClass;
```




<hr>



### variable DeviceIndex 

```C++
int32 FTryllMemoryComponent::DeviceIndex;
```



GPU index 0 is valid; host RAM is -1. Do not treat 0 as missing. 


        

<hr>



### variable EstimateBasis 

```C++
FString FTryllMemoryComponent::EstimateBasis;
```




<hr>



### variable Kind 

```C++
FString FTryllMemoryComponent::Kind;
```




<hr>



### variable LastTurnAtMs 

```C++
int64 FTryllMemoryComponent::LastTurnAtMs;
```




<hr>



### variable Name 

```C++
FString FTryllMemoryComponent::Name;
```




<hr>



### variable NodeName 

```C++
FString FTryllMemoryComponent::NodeName;
```




<hr>



### variable Owner 

```C++
ETryllMemoryOwnerKind FTryllMemoryComponent::Owner;
```




<hr>



### variable PerDevice 

```C++
TArray<FTryllMemoryComponentDevice> FTryllMemoryComponent::PerDevice;
```




<hr>



### variable RamBytes 

```C++
int64 FTryllMemoryComponent::RamBytes;
```




<hr>



### variable RamBytesHigh 

```C++
int64 FTryllMemoryComponent::RamBytesHigh;
```




<hr>



### variable SessionId 

```C++
int64 FTryllMemoryComponent::SessionId;
```




<hr>



### variable Tier 

```C++
ETryllMemoryTier FTryllMemoryComponent::Tier;
```




<hr>



### variable UsedTokens 

```C++
int32 FTryllMemoryComponent::UsedTokens;
```




<hr>



### variable VramBytes 

```C++
int64 FTryllMemoryComponent::VramBytes;
```




<hr>



### variable bOccupancyStale 

```C++
bool FTryllMemoryComponent::bOccupancyStale;
```




<hr>



### variable bResident 

```C++
bool FTryllMemoryComponent::bResident;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/unreal/Source/TryllClient/Public/TryllMemorySnapshot.h`

