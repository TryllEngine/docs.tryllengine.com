

# Struct UTryllSubsystem::FPendingEmbeddedStorage



[**ClassList**](annotated.md) **>** [**FPendingEmbeddedStorage**](struct_u_tryll_subsystem_1_1_f_pending_embedded_storage.md)


























## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Name**](#variable-name)  <br> |
|  TFunction&lt; FTryllOnEmbeddedStorageCreated &gt; | [**OnCreateComplete**](#variable-oncreatecomplete)  <br> |
|  TFunction&lt; FTryllOnEmbeddedStorageDestroyed &gt; | [**OnDestroyComplete**](#variable-ondestroycomplete)  <br> |
|  std::uint64\_t | [**RequestId**](#variable-requestid)   = `0`<br> |
|  bool | [**bIsCreate**](#variable-biscreate)   = `true`<br> |












































## Public Attributes Documentation




### variable Name 

```C++
FString UTryllSubsystem::FPendingEmbeddedStorage::Name;
```




<hr>



### variable OnCreateComplete 

```C++
TFunction<FTryllOnEmbeddedStorageCreated> UTryllSubsystem::FPendingEmbeddedStorage::OnCreateComplete;
```




<hr>



### variable OnDestroyComplete 

```C++
TFunction<FTryllOnEmbeddedStorageDestroyed> UTryllSubsystem::FPendingEmbeddedStorage::OnDestroyComplete;
```




<hr>



### variable RequestId 

```C++
std::uint64_t UTryllSubsystem::FPendingEmbeddedStorage::RequestId;
```




<hr>



### variable bIsCreate 

```C++
bool UTryllSubsystem::FPendingEmbeddedStorage::bIsCreate;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/tryll-mono/server/client-unreal/Source/TryllClient/Public/TryllSubsystem.h`

