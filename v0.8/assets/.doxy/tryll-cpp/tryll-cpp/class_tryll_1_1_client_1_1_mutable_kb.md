

# Class Tryll::Client::MutableKb



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**MutableKb**](class_tryll_1_1_client_1_1_mutable_kb.md)



_Experimental session-scoped mutable knowledge-base handle._ 

* `#include <MutableKb.h>`





































## Public Functions

| Type | Name |
| ---: | :--- |
|  std::uint32\_t | [**Clear**](#function-clear) (std::chrono::milliseconds timeout=std::chrono::seconds{10}) <br> |
|  std::future&lt; std::uint32\_t &gt; | [**ClearAsync**](#function-clearasync) () <br> |
|  std::uint32\_t | [**Delete**](#function-delete) (std::span&lt; const std::string &gt; ids, std::chrono::milliseconds timeout=std::chrono::seconds{10}) <br> |
|  std::future&lt; std::uint32\_t &gt; | [**DeleteAsync**](#function-deleteasync) (std::span&lt; const std::string &gt; ids) <br> |
|  [**MutableKbSnapshot**](struct_tryll_1_1_client_1_1_mutable_kb_snapshot.md) | [**Export**](#function-export) (std::chrono::milliseconds timeout=std::chrono::seconds{30}) <br> |
|  std::future&lt; [**MutableKbSnapshot**](struct_tryll_1_1_client_1_1_mutable_kb_snapshot.md) &gt; | [**ExportAsync**](#function-exportasync) () <br> |
|  std::optional&lt; [**MutableKbRecord**](struct_tryll_1_1_client_1_1_mutable_kb_record.md) &gt; | [**Get**](#function-get) (std::string\_view id, std::chrono::milliseconds timeout=std::chrono::seconds{10}) <br> |
|  std::future&lt; std::optional&lt; [**MutableKbRecord**](struct_tryll_1_1_client_1_1_mutable_kb_record.md) &gt; &gt; | [**GetAsync**](#function-getasync) (std::string\_view id) <br> |
|  [**MutableKbImportResult**](struct_tryll_1_1_client_1_1_mutable_kb_import_result.md) | [**Import**](#function-import) (const [**MutableKbSnapshot**](struct_tryll_1_1_client_1_1_mutable_kb_snapshot.md) & snapshot, MutableKbCompatibility compatibility=MutableKbCompatibility::Strict, std::chrono::milliseconds timeout=std::chrono::seconds{60}) <br> |
|  std::future&lt; [**MutableKbImportResult**](struct_tryll_1_1_client_1_1_mutable_kb_import_result.md) &gt; | [**ImportAsync**](#function-importasync) (const [**MutableKbSnapshot**](struct_tryll_1_1_client_1_1_mutable_kb_snapshot.md) & snapshot, MutableKbCompatibility compatibility=MutableKbCompatibility::Strict) <br> |
|  bool | [**IsValid**](#function-isvalid) () noexcept const<br> |
|  [**MutableKbListPage**](struct_tryll_1_1_client_1_1_mutable_kb_list_page.md) | [**List**](#function-list) (std::uint32\_t offset=0, std::uint32\_t limit=50, std::chrono::milliseconds timeout=std::chrono::seconds{10}) <br> |
|  std::future&lt; [**MutableKbListPage**](struct_tryll_1_1_client_1_1_mutable_kb_list_page.md) &gt; | [**ListAsync**](#function-listasync) (std::uint32\_t offset=0, std::uint32\_t limit=50) <br> |
|   | [**MutableKb**](#function-mutablekb-12) () = default<br> |
|  const std::string & | [**Name**](#function-name) () noexcept const<br> |
|  std::vector&lt; [**MutableKbHit**](struct_tryll_1_1_client_1_1_mutable_kb_hit.md) &gt; | [**Query**](#function-query) (std::string\_view queryText, std::uint32\_t topK=4, ::Tryll::RetrievalMode mode=::Tryll::RetrievalMode\_Dense, std::chrono::milliseconds timeout=std::chrono::seconds{30}) <br> |
|  std::future&lt; std::vector&lt; [**MutableKbHit**](struct_tryll_1_1_client_1_1_mutable_kb_hit.md) &gt; &gt; | [**QueryAsync**](#function-queryasync) (std::string\_view queryText, std::uint32\_t topK=4, ::Tryll::RetrievalMode mode=::Tryll::RetrievalMode\_Dense) <br> |
|  const std::string & | [**Schema**](#function-schema) () noexcept const<br> |
|  [**MutableKbStats**](struct_tryll_1_1_client_1_1_mutable_kb_stats.md) | [**Stats**](#function-stats) (std::chrono::milliseconds timeout=std::chrono::seconds{10}) <br> |
|  std::future&lt; [**MutableKbStats**](struct_tryll_1_1_client_1_1_mutable_kb_stats.md) &gt; | [**StatsAsync**](#function-statsasync) () <br> |
|  std::vector&lt; [**MutableKbWriteResult**](struct_tryll_1_1_client_1_1_mutable_kb_write_result.md) &gt; | [**Write**](#function-write) (std::span&lt; const [**MutableKbUpsert**](struct_tryll_1_1_client_1_1_mutable_kb_upsert.md) &gt; records, std::chrono::milliseconds timeout=std::chrono::seconds{30}) <br> |
|  std::future&lt; std::vector&lt; [**MutableKbWriteResult**](struct_tryll_1_1_client_1_1_mutable_kb_write_result.md) &gt; &gt; | [**WriteAsync**](#function-writeasync) (std::span&lt; const [**MutableKbUpsert**](struct_tryll_1_1_client_1_1_mutable_kb_upsert.md) &gt; records) <br> |




























## Public Functions Documentation




### function Clear 

```C++
std::uint32_t Tryll::Client::MutableKb::Clear (
    std::chrono::milliseconds timeout=std::chrono::seconds{10}
) 
```




<hr>



### function ClearAsync 

```C++
std::future< std::uint32_t > Tryll::Client::MutableKb::ClearAsync () 
```




<hr>



### function Delete 

```C++
std::uint32_t Tryll::Client::MutableKb::Delete (
    std::span< const std::string > ids,
    std::chrono::milliseconds timeout=std::chrono::seconds{10}
) 
```




<hr>



### function DeleteAsync 

```C++
std::future< std::uint32_t > Tryll::Client::MutableKb::DeleteAsync (
    std::span< const std::string > ids
) 
```




<hr>



### function Export 

```C++
MutableKbSnapshot Tryll::Client::MutableKb::Export (
    std::chrono::milliseconds timeout=std::chrono::seconds{30}
) 
```




<hr>



### function ExportAsync 

```C++
std::future< MutableKbSnapshot > Tryll::Client::MutableKb::ExportAsync () 
```




<hr>



### function Get 

```C++
std::optional< MutableKbRecord > Tryll::Client::MutableKb::Get (
    std::string_view id,
    std::chrono::milliseconds timeout=std::chrono::seconds{10}
) 
```




<hr>



### function GetAsync 

```C++
std::future< std::optional< MutableKbRecord > > Tryll::Client::MutableKb::GetAsync (
    std::string_view id
) 
```




<hr>



### function Import 

```C++
MutableKbImportResult Tryll::Client::MutableKb::Import (
    const MutableKbSnapshot & snapshot,
    MutableKbCompatibility compatibility=MutableKbCompatibility::Strict,
    std::chrono::milliseconds timeout=std::chrono::seconds{60}
) 
```




<hr>



### function ImportAsync 

```C++
std::future< MutableKbImportResult > Tryll::Client::MutableKb::ImportAsync (
    const MutableKbSnapshot & snapshot,
    MutableKbCompatibility compatibility=MutableKbCompatibility::Strict
) 
```




<hr>



### function IsValid 

```C++
inline bool Tryll::Client::MutableKb::IsValid () noexcept const
```




<hr>



### function List 

```C++
MutableKbListPage Tryll::Client::MutableKb::List (
    std::uint32_t offset=0,
    std::uint32_t limit=50,
    std::chrono::milliseconds timeout=std::chrono::seconds{10}
) 
```




<hr>



### function ListAsync 

```C++
std::future< MutableKbListPage > Tryll::Client::MutableKb::ListAsync (
    std::uint32_t offset=0,
    std::uint32_t limit=50
) 
```




<hr>



### function MutableKb [1/2]

```C++
Tryll::Client::MutableKb::MutableKb () = default
```




<hr>



### function Name 

```C++
inline const std::string & Tryll::Client::MutableKb::Name () noexcept const
```




<hr>



### function Query 

```C++
std::vector< MutableKbHit > Tryll::Client::MutableKb::Query (
    std::string_view queryText,
    std::uint32_t topK=4,
    ::Tryll::RetrievalMode mode=::Tryll::RetrievalMode_Dense,
    std::chrono::milliseconds timeout=std::chrono::seconds{30}
) 
```




<hr>



### function QueryAsync 

```C++
std::future< std::vector< MutableKbHit > > Tryll::Client::MutableKb::QueryAsync (
    std::string_view queryText,
    std::uint32_t topK=4,
    ::Tryll::RetrievalMode mode=::Tryll::RetrievalMode_Dense
) 
```




<hr>



### function Schema 

```C++
inline const std::string & Tryll::Client::MutableKb::Schema () noexcept const
```




<hr>



### function Stats 

```C++
MutableKbStats Tryll::Client::MutableKb::Stats (
    std::chrono::milliseconds timeout=std::chrono::seconds{10}
) 
```




<hr>



### function StatsAsync 

```C++
std::future< MutableKbStats > Tryll::Client::MutableKb::StatsAsync () 
```




<hr>



### function Write 

```C++
std::vector< MutableKbWriteResult > Tryll::Client::MutableKb::Write (
    std::span< const MutableKbUpsert > records,
    std::chrono::milliseconds timeout=std::chrono::seconds{30}
) 
```




<hr>



### function WriteAsync 

```C++
std::future< std::vector< MutableKbWriteResult > > Tryll::Client::MutableKb::WriteAsync (
    std::span< const MutableKbUpsert > records
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/cpp/include/tryll/MutableKb.h`

