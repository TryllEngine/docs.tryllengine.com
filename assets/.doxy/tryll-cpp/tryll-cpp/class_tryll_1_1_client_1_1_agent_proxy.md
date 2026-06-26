

# Class Tryll::Client::AgentProxy



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**AgentProxy**](class_tryll_1_1_client_1_1_agent_proxy.md)



_Client-side handle for one server-side agent._ [More...](#detailed-description)

* `#include <AgentProxy.h>`

















## Public Types

| Type | Name |
| ---: | :--- |
| typedef std::function&lt; void(std::string\_view text, bool isDelta, bool isFinal)&gt; | [**AnswerTextCallback**](#typedef-answertextcallback)  <br>_Callback type for streaming text chunks._  |
| typedef std::function&lt; void(const [**TryllError**](class_tryll_1_1_client_1_1_tryll_error.md) &error)&gt; | [**ErrorCallback**](#typedef-errorcallback)  <br>_Callback type for agent-level error notifications._  |
| typedef std::function&lt; void(std::string\_view intent, std::string\_view recordId, std::size\_t recordIndex, float distance)&gt; | [**IntentClassifiedCallback**](#typedef-intentclassifiedcallback)  <br>_Callback type for intent-classification notifications._  |
| typedef std::function&lt; void(std::string\_view nodeName, std::string\_view eventType, const std::vector&lt; [**NodeEventKeyValue**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-nodeeventkeyvalue) &gt; &kvPairs)&gt; | [**NodeEventCallback**](#typedef-nodeeventcallback)  <br>_Generic_ `NodeEvent` _callback._ |
| typedef std::pair&lt; std::string, std::string &gt; | [**NodeEventKeyValue**](#typedef-nodeeventkeyvalue)  <br>_One ordered key/value pair in a_ `NodeEvent` _payload._ |
| typedef std::function&lt; void(std::string\_view toolName, std::string\_view argumentsJson)&gt; | [**ToolCallCallback**](#typedef-toolcallcallback)  <br>_Callback type for tool-call notifications._  |
| typedef std::function&lt; void(std::span&lt; const int16\_t &gt; pcm)&gt; | [**TtsAudioCallback**](#typedef-ttsaudiocallback)  <br>_Callback type for streaming TTS PCM chunks._  |
| typedef std::function&lt; void(uint32\_t sampleRate, uint16\_t channels, uint16\_t bitsPerSample)&gt; | [**TtsAudioFormatCallback**](#typedef-ttsaudioformatcallback)  <br>_Callback type for the TTS audio format announcement._  |
| typedef std::function&lt; void(::Tryll::TurnStatus status, std::string\_view debugInfoJson, std::int32\_t tokensGenerated)&gt; | [**TurnCompleteCallback**](#typedef-turncompletecallback)  <br>_Callback type for turn-complete notifications._  |




















## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**AgentProxy**](#function-agentproxy-12) () = default<br> |
|  void | [**ChangeParams**](#function-changeparams) (std::string\_view nodeName, [**NodeParamsVariant**](namespace_tryll_1_1_client.md#typedef-nodeparamsvariant) params) <br>_Blocking convenience wrapper over_ [_**ChangeParamsAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-changeparamsasync) _._ |
|  std::future&lt; void &gt; | [**ChangeParamsAsync**](#function-changeparamsasync) (std::string\_view nodeName, [**NodeParamsVariant**](namespace_tryll_1_1_client.md#typedef-nodeparamsvariant) params) <br>_Apply typed node parameters to a named workflow node asynchronously._  |
|  void | [**Destroy**](#function-destroy) () <br>_Blocking convenience wrapper over_ [_**DestroyAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-destroyasync) _._ |
|  std::future&lt; void &gt; | [**DestroyAsync**](#function-destroyasync) () <br>_Request agent destruction on the server asynchronously._  |
|  std::uint64\_t | [**GetAgentId**](#function-getagentid) () noexcept const<br>_Server-assigned agent identifier._  |
|  void | [**SendText**](#function-sendtext) (std::string\_view text) <br>_Send a text message to the agent (fire-and-forget)._  |
|  void | [**SetOnAnswerText**](#function-setonanswertext) ([**AnswerTextCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-answertextcallback) cb) <br>_Register a callback for streaming text chunks._  |
|  void | [**SetOnError**](#function-setonerror) ([**ErrorCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-errorcallback) cb) <br>_Register a callback for agent-level error notifications._  |
|  void | [**SetOnIntentClassified**](#function-setonintentclassified) ([**IntentClassifiedCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-intentclassifiedcallback) cb) <br>_Register a callback for intent-classification notifications._  |
|  void | [**SetOnNodeEvent**](#function-setonnodeevent) ([**NodeEventCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-nodeeventcallback) cb) <br>_Register a generic_ `NodeEvent` _fallback callback._ |
|  void | [**SetOnToolCall**](#function-setontoolcall) ([**ToolCallCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-toolcallcallback) cb) <br>_Register a callback for tool-call notifications._  |
|  void | [**SetOnTtsAudio**](#function-setonttsaudio) ([**TtsAudioCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-ttsaudiocallback) cb) <br>_Register (or clear) the TTS audio chunk callback._  |
|  void | [**SetOnTtsAudioFormat**](#function-setonttsaudioformat) ([**TtsAudioFormatCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-ttsaudioformatcallback) cb) <br>_Register (or clear) the TTS audio format callback._  |
|  void | [**SetOnTurnComplete**](#function-setonturncomplete) ([**TurnCompleteCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-turncompletecallback) cb) <br>_Register a callback for turn-complete notifications._  |




























## Detailed Description


## Public Types Documentation




### typedef AnswerTextCallback 

_Callback type for streaming text chunks._ 
```C++
using Tryll::Client::AgentProxy::AnswerTextCallback = 
std::function<void(std::string_view text, bool isDelta, bool isFinal)>;
```



Invoked on the reader thread for each `AnswerText` frame received from the server (including frames from server-initiated turns such as voice autosend). Must return quickly and must not call any blocking [**TryllClient**](class_tryll_1_1_client_1_1_tryll_client.md) or [**AgentProxy**](class_tryll_1_1_client_1_1_agent_proxy.md) methods.


Parameters:
* `text` — streamed text chunk.
* `isDelta` — true when `text` is an incremental token/chunk; false when it is the accumulated response so far.
* `isFinal` — true for the last chunk before `TurnComplete`. 




        

<hr>



### typedef ErrorCallback 

_Callback type for agent-level error notifications._ 
```C++
using Tryll::Client::AgentProxy::ErrorCallback = std::function<void(const TryllError& error)>;
```



Invoked on the reader thread when a `SendText` turn is terminated by a server-reported `ErrorResponse`, or when the connection drops mid-turn. Must return quickly and must not call any blocking methods.




**Parameters:**


* `error` The error reported by the server or transport layer. 




        

<hr>



### typedef IntentClassifiedCallback 

_Callback type for intent-classification notifications._ 
```C++
using Tryll::Client::AgentProxy::IntentClassifiedCallback = 
std::function<void(std::string_view intent,
                   std::string_view recordId,
                   std::size_t      recordIndex,
                   float            distance)>;
```



Invoked on the reader thread for each `NodeEvent` with `event_type="intent_classified"` — fired by `ClassifyIntentNode` on its "found" path when `notify_client="true"`.


Parameters:
* `intent` — the classified intent label.
* `recordId` — id of the top-1 matched KB record.
* `recordIndex` — 0-based index of the matched record in storage.
* `distance` — cosine distance of the match.




Must return quickly and must not call any blocking methods. 


        

<hr>



### typedef NodeEventCallback 

_Generic_ `NodeEvent` _callback._
```C++
using Tryll::Client::AgentProxy::NodeEventCallback = 
std::function<void(std::string_view nodeName,
                   std::string_view eventType,
                   const std::vector<NodeEventKeyValue>& kvPairs)>;
```



Invoked on the reader thread when a `NodeEvent` arrives whose `event_type` is unrecognised or whose typed callback is not set. If a typed callback (e.g. [**ToolCallCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-toolcallcallback), [**IntentClassifiedCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-intentclassifiedcallback)) is registered for the incoming `event_type`, this callback is NOT fired for that event.


Parameters:
* `nodeName` — name of the emitting workflow node.
* `eventType` — event discriminator.
* `kvPairs` — ordered string→string payload. 




        

<hr>



### typedef NodeEventKeyValue 

_One ordered key/value pair in a_ `NodeEvent` _payload._
```C++
using Tryll::Client::AgentProxy::NodeEventKeyValue = std::pair<std::string, std::string>;
```




<hr>



### typedef ToolCallCallback 

_Callback type for tool-call notifications._ 
```C++
using Tryll::Client::AgentProxy::ToolCallCallback = 
std::function<void(std::string_view toolName, std::string_view argumentsJson)>;
```



Invoked on the reader thread for each `NodeEvent` with `event_type="tool_call"` received from the server (i.e. when the agent's graph has a `ToolCall` node with `notify_client="true"`).


Parameters:
* `toolName` — the tool the model asked to call.
* `argumentsJson` — compact JSON object of arguments, e.g. `{"city"`:"Berlin"}.




Must return quickly and must not call any blocking methods. 


        

<hr>



### typedef TtsAudioCallback 

_Callback type for streaming TTS PCM chunks._ 
```C++
using Tryll::Client::AgentProxy::TtsAudioCallback = 
std::function<void(std::span<const int16_t> pcm)>;
```



Fired for each int16 LE PCM chunk synthesized by a GenerateAndSpeak node. End-of-stream is signalled by the existing turn-complete future returned from SendMessage — there is no per-chunk final flag.


Must return quickly and must not call any blocking methods. 


        

<hr>



### typedef TtsAudioFormatCallback 

_Callback type for the TTS audio format announcement._ 
```C++
using Tryll::Client::AgentProxy::TtsAudioFormatCallback = 
std::function<void(uint32_t sampleRate, uint16_t channels, uint16_t bitsPerSample)>;
```



Fired once per GenerateAndSpeak turn, before the first TtsAudioFrame.


Parameters:
* `sampleRate` — PCM sample rate (e.g. 22050).
* `channels` — channel count (always 1 for current models).
* `bitsPerSample` — bit depth (always 16). 




        

<hr>



### typedef TurnCompleteCallback 

_Callback type for turn-complete notifications._ 
```C++
using Tryll::Client::AgentProxy::TurnCompleteCallback = 
std::function<void(::Tryll::TurnStatus status,
                   std::string_view debugInfoJson,
                   std::int32_t tokensGenerated)>;
```



Invoked on the reader thread once per turn, immediately after `TurnComplete` arrives. Covers both client-initiated turns (via [**SendText**](class_tryll_1_1_client_1_1_agent_proxy.md#function-sendtext)) and server-initiated turns (voice autosend).


Parameters:
* `status` — turn outcome (::Tryll::TurnStatus).
* `debugInfoJson` — JSON diagnostics string; non-empty only when the agent was created with `enableDiagnostics=true`.
* `tokensGenerated` — total tokens sampled across all generation nodes.




Must return quickly and must not call any blocking methods. 


        

<hr>
## Public Functions Documentation




### function AgentProxy [1/2]

```C++
Tryll::Client::AgentProxy::AgentProxy () = default
```




<hr>



### function ChangeParams 

_Blocking convenience wrapper over_ [_**ChangeParamsAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-changeparamsasync) _._
```C++
void Tryll::Client::AgentProxy::ChangeParams (
    std::string_view nodeName,
    NodeParamsVariant params
) 
```





**Exception:**


* `TryllError` On server-reported errors or disconnect. 




        

<hr>



### function ChangeParamsAsync 

_Apply typed node parameters to a named workflow node asynchronously._ 
```C++
std::future< void > Tryll::Client::AgentProxy::ChangeParamsAsync (
    std::string_view nodeName,
    NodeParamsVariant params
) 
```



The supplied `params` must match the concrete type of the target node (type mismatch → `InvalidParamValue` 3007 from the server). The mutation diff-loop on the server vetoes structural fields and invokes OnXxxChanged callbacks for fields that have changed.


The C++ client does not cache a baseline copy of authored params: callers must own the typed params they last sent (or authored at `CreateAgent`) and start each mutation from that local copy. Structural fields must match the create-time values or the server returns `ParamNotMutable`.


Typical clone-set-send idiom: 
```C++
GenerateParamsT p = myBaseline;   // caller-owned copy of authored params
p.system_prompt    = "New system prompt";
agent.ChangeParams("gen", std::move(p));
```





**Parameters:**


* `nodeName` Instance name of the target node (as declared in the graph). 
* `params` Whole authored typed params for the node.



**Returns:**

Future completing on `Ack`. Throws [**TryllError**](class_tryll_1_1_client_1_1_tryll_error.md) on server-reported errors (UnknownNode 3005, ParamNotMutable 3006, InvalidParamValue 3007, or AgentBusy 3004). 





        

<hr>



### function Destroy 

_Blocking convenience wrapper over_ [_**DestroyAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-destroyasync) _._
```C++
void Tryll::Client::AgentProxy::Destroy () 
```





**Exception:**


* `TryllError` On server-reported errors, disconnect, or timeout (configured by the owning client). 




        

<hr>



### function DestroyAsync 

_Request agent destruction on the server asynchronously._ 
```C++
std::future< void > Tryll::Client::AgentProxy::DestroyAsync () 
```



All registered callbacks are cleared before the destroy request is sent. The proxy becomes inert once the returned future completes; further `SendText` calls are no-ops.




**Returns:**

Future completing when the server acknowledges destruction. 





        

<hr>



### function GetAgentId 

_Server-assigned agent identifier._ 
```C++
inline std::uint64_t Tryll::Client::AgentProxy::GetAgentId () noexcept const
```





**Returns:**

The `agent_id` field from the original `CreateAgentResponse`. 





        

<hr>



### function SendText 

_Send a text message to the agent (fire-and-forget)._ 
```C++
void Tryll::Client::AgentProxy::SendText (
    std::string_view text
) 
```



Enqueues the message and returns immediately. The response arrives through the persistent callbacks registered via [**SetOnAnswerText**](class_tryll_1_1_client_1_1_agent_proxy.md#function-setonanswertext), [**SetOnTurnComplete**](class_tryll_1_1_client_1_1_agent_proxy.md#function-setonturncomplete), and [**SetOnError**](class_tryll_1_1_client_1_1_agent_proxy.md#function-setonerror). Register those callbacks before the first `SendText` call.




**Parameters:**


* `text` User-turn text to send. 




        

<hr>



### function SetOnAnswerText 

_Register a callback for streaming text chunks._ 
```C++
void Tryll::Client::AgentProxy::SetOnAnswerText (
    AnswerTextCallback cb
) 
```



Replaces any previously set callback. Pass an empty (default- constructed) [**AnswerTextCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-answertextcallback) to unregister.


Fires on the reader thread for every `AnswerText` frame, including frames from server-initiated turns (e.g. voice autosend).




**Parameters:**


* `cb` Callable invoked with ``(text, isDelta, isFinal), or empty to clear. 




        

<hr>



### function SetOnError 

_Register a callback for agent-level error notifications._ 
```C++
void Tryll::Client::AgentProxy::SetOnError (
    ErrorCallback cb
) 
```



Replaces any previously set callback. Pass an empty (default- constructed) [**ErrorCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-errorcallback) to unregister.


Fires on the reader thread when a `SendText` turn is terminated by a server-reported error or a connection drop. Does not fire for ::Tryll::TurnStatus\_Error turns (those arrive via [**SetOnTurnComplete**](class_tryll_1_1_client_1_1_agent_proxy.md#function-setonturncomplete)).




**Parameters:**


* `cb` Callable invoked with the [**TryllError**](class_tryll_1_1_client_1_1_tryll_error.md), or empty to clear. 




        

<hr>



### function SetOnIntentClassified 

_Register a callback for intent-classification notifications._ 
```C++
void Tryll::Client::AgentProxy::SetOnIntentClassified (
    IntentClassifiedCallback cb
) 
```



Replaces any previously set callback. Pass an empty (default- constructed) [**IntentClassifiedCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-intentclassifiedcallback) to unregister.


Fires on the reader thread for every `NodeEvent` with `event_type="intent_classified"` the server sends for this agent (emitted by `ClassifyIntentNode` when `notify_client="true"`).




**Parameters:**


* `cb` Callable invoked with ``(intent, recordId, recordIndex, distance), or empty to clear. 




        

<hr>



### function SetOnNodeEvent 

_Register a generic_ `NodeEvent` _fallback callback._
```C++
void Tryll::Client::AgentProxy::SetOnNodeEvent (
    NodeEventCallback cb
) 
```



Replaces any previously set callback. Pass an empty (default-constructed) [**NodeEventCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-nodeeventcallback) to unregister.


Fires on the reader thread for every `NodeEvent` whose `event_type` is either unrecognised or has no typed callback registered. When a typed callback handles the event, this one is NOT invoked for the same event.




**Parameters:**


* `cb` Callable invoked with ``(nodeName, eventType, kvPairs), or empty to clear. 




        

<hr>



### function SetOnToolCall 

_Register a callback for tool-call notifications._ 
```C++
void Tryll::Client::AgentProxy::SetOnToolCall (
    ToolCallCallback cb
) 
```



Replaces any previously set callback. Pass an empty (default- constructed) [**ToolCallCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-toolcallcallback) to unregister.


Fires on the reader thread for every `NodeEvent` with `event_type="tool_call"` the server sends for this agent.




**Parameters:**


* `cb` Callable invoked with ``(toolName, argumentsJson), or empty to clear. 




        

<hr>



### function SetOnTtsAudio 

_Register (or clear) the TTS audio chunk callback._ 
```C++
void Tryll::Client::AgentProxy::SetOnTtsAudio (
    TtsAudioCallback cb
) 
```




<hr>



### function SetOnTtsAudioFormat 

_Register (or clear) the TTS audio format callback._ 
```C++
void Tryll::Client::AgentProxy::SetOnTtsAudioFormat (
    TtsAudioFormatCallback cb
) 
```




<hr>



### function SetOnTurnComplete 

_Register a callback for turn-complete notifications._ 
```C++
void Tryll::Client::AgentProxy::SetOnTurnComplete (
    TurnCompleteCallback cb
) 
```



Replaces any previously set callback. Pass an empty (default- constructed) [**TurnCompleteCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-turncompletecallback) to unregister.


Fires on the reader thread once per `TurnComplete` frame, for both client-initiated and server-initiated turns.




**Parameters:**


* `cb` Callable invoked with ``(status, debugInfoJson, tokensGenerated), or empty to clear. 




        

<hr>## Friends Documentation





### friend TryllClientImpl 

```C++
class Tryll::Client::AgentProxy::TryllClientImpl (
    Internal::TryllClientImpl
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/cpp/include/tryll/AgentProxy.h`

