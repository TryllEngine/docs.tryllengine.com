

# Struct FTryllNodeDescription



[**ClassList**](annotated.md) **>** [**FTryllNodeDescription**](struct_f_tryll_node_description.md)



[More...](#detailed-description)

* `#include <TryllGraphDescription.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  FString | [**Name**](#variable-name)  <br> |
|  TObjectPtr&lt; [**UTryllNodeParamsBase**](class_u_tryll_node_params_base.md) &gt; | [**Params**](#variable-params)  <br> |












































## Detailed Description


Description of one workflow node sent to the server. 


    
## Public Attributes Documentation




### variable Name 

```C++
FString FTryllNodeDescription::Name;
```




<hr>



### variable Params 

```C++
TObjectPtr<UTryllNodeParamsBase> FTryllNodeDescription::Params;
```



Typed node parameters. Use UTryllNodeParamsFactory::MakeXxxParams() to create; use Instanced so the UObject is serialised inline in Blueprint/editor assets. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/TryllGraphDescription.h`

