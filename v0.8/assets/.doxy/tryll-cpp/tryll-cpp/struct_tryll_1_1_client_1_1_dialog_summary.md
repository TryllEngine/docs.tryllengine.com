

# Struct Tryll::Client::DialogSummary



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**DialogSummary**](struct_tryll_1_1_client_1_1_dialog_summary.md)



[More...](#detailed-description)

* `#include <DialogSnapshot.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  std::uint32\_t | [**coveredThrough**](#variable-coveredthrough)   = `0`<br> |
|  std::string | [**producer**](#variable-producer)  <br> |
|  std::string | [**text**](#variable-text)  <br> |












































## Detailed Description


Rolling conversation summary written by a Summarize node. `coveredThrough` is the index of the last interaction folded into `text`, inclusive; the server rejects an import whose value is at or past the interaction count. 


    
## Public Attributes Documentation




### variable coveredThrough 

```C++
std::uint32_t Tryll::Client::DialogSummary::coveredThrough;
```




<hr>



### variable producer 

```C++
std::string Tryll::Client::DialogSummary::producer;
```




<hr>



### variable text 

```C++
std::string Tryll::Client::DialogSummary::text;
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/cpp/include/tryll/DialogSnapshot.h`

