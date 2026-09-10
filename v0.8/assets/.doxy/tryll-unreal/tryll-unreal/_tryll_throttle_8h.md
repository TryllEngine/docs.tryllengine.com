

# File TryllThrottle.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllThrottle.h**](_tryll_throttle_8h.md)





* `#include "CoreMinimal.h"`
* `#include "TryllThrottle.generated.h"`

















## Public Types

| Type | Name |
| ---: | :--- |
| enum uint8 | [**ETryllWorkload**](#enum-etryllworkload)  <br> |
















































## Public Types Documentation




### enum ETryllWorkload 

```C++
enum ETryllWorkload {
    Interactive,
    Background
};
```



Whether a human ever waits on an agent's turns. Purely a scheduling hint for the inference budget controller — it never changes node semantics. Set Background for world-logic/evaluator agents so their work is paced or deferred first under GPU pressure. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllThrottle.h`

