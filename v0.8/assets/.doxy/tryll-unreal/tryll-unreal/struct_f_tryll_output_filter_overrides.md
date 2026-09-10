

# Struct FTryllOutputFilterOverrides



[**ClassList**](annotated.md) **>** [**FTryllOutputFilterOverrides**](struct_f_tryll_output_filter_overrides.md)



[More...](#detailed-description)

* `#include <TryllOutputFilterOverrides.h>`





















## Public Attributes

| Type | Name |
| ---: | :--- |
|  int32 | [**MaxPrefixBytes**](#variable-maxprefixbytes)   = `16`<br> |
|  int32 | [**MaxSpanBytes**](#variable-maxspanbytes)   = `64`<br> |
|  bool | [**bStripAsteriskSpans**](#variable-bstripasteriskspans)   = `false`<br> |
|  bool | [**bStripBracketSpans**](#variable-bstripbracketspans)   = `false`<br> |
|  bool | [**bStripParenSpans**](#variable-bstripparenspans)   = `false`<br> |
|  bool | [**bStripSpeakerPrefix**](#variable-bstripspeakerprefix)   = `false`<br> |












































## Detailed Description


Bounded streaming cleanup of roleplay artifacts in Generate / GenerateAndSpeak output (speaker prefixes and paired markup spans). All flags default off. Budgets are UTF-8 byte counts. Missing wire sub-table is treated by the server as an explicit all-default/off table; generated clients still materialize the sub-table (same invariant as SamplingOverrides). No on\_change — nodes read live params at Execute. 


    
## Public Attributes Documentation




### variable MaxPrefixBytes 

```C++
int32 FTryllOutputFilterOverrides::MaxPrefixBytes;
```



Max UTF-8 bytes scanned for a speaker prefix. Default 16. 


        

<hr>



### variable MaxSpanBytes 

```C++
int32 FTryllOutputFilterOverrides::MaxSpanBytes;
```



Max UTF-8 bytes held while matching a span. Default 64; shared across pairs. 


        

<hr>



### variable bStripAsteriskSpans 

```C++
bool FTryllOutputFilterOverrides::bStripAsteriskSpans;
```



Delete `*…*` spans (first-close wins; no nesting). 


        

<hr>



### variable bStripBracketSpans 

```C++
bool FTryllOutputFilterOverrides::bStripBracketSpans;
```



Delete `[…]` spans (first-close wins; no nesting). 


        

<hr>



### variable bStripParenSpans 

```C++
bool FTryllOutputFilterOverrides::bStripParenSpans;
```



Delete `(…)` spans (first-close wins; no nesting). 


        

<hr>



### variable bStripSpeakerPrefix 

```C++
bool FTryllOutputFilterOverrides::bStripSpeakerPrefix;
```



Strip a leading `Name: ` speaker tag once per turn (ASCII heuristic). 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/Generated/Nodes/TryllOutputFilterOverrides.h`

