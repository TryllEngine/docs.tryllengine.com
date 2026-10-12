

# Class Tryll::Client::AgentProxy



[**ClassList**](annotated.md) **>** [**Tryll**](namespace_tryll.md) **>** [**Client**](namespace_tryll_1_1_client.md) **>** [**AgentProxy**](class_tryll_1_1_client_1_1_agent_proxy.md)



_Client-side handle for one server-side agent._ [More...](#detailed-description)

* `#include <AgentProxy.h>`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**DialogInteraction**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_dialog_interaction.md) <br> |
| struct | [**IntentClassifiedEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_intent_classified_event.md) <br>_Owning payload for_ `event_type="intent_classified"` _(embedding classifier)._ |
| struct | [**IntentLlmClassifiedEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_intent_llm_classified_event.md) <br>_Owning payload for_ `event_type="intent_llm_classified"` _._ |
| struct | [**IntentLlmLabelProb**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_intent_llm_label_prob.md) <br>_One label / first-token probability pair from ClassifyIntentLLM._  |
| struct | [**ToolCallEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_tool_call_event.md) <br>_Owning tool-call event for deferred / async handling._  |
| struct | [**ToolResult**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_tool_result.md) <br>_One model-visible result correlated to a tool\_call event's call\_id._  |
| struct | [**VisemeEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_viseme_event.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| typedef std::function&lt; void(std::string\_view nodeName, std::string\_view text, bool isDelta, bool isFinal)&gt; | [**AnswerTextCallback**](#typedef-answertextcallback)  <br>_Callback type for streaming text chunks._  |
| typedef std::function&lt; void(const [**TryllError**](class_tryll_1_1_client_1_1_tryll_error.md) &error)&gt; | [**ErrorCallback**](#typedef-errorcallback)  <br>_Callback type for agent-level error notifications._  |
| typedef std::function&lt; void(const [**IntentClassifiedEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_intent_classified_event.md) &event)&gt; | [**IntentClassifiedCallback**](#typedef-intentclassifiedcallback)  <br>_Callback type for embedding intent-classification notifications._  |
| typedef std::function&lt; void(const [**IntentLlmClassifiedEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_intent_llm_classified_event.md) &event)&gt; | [**IntentLlmClassifiedCallback**](#typedef-intentllmclassifiedcallback)  <br>_Callback type for LLM intent-classification notifications._  |
| typedef std::function&lt; void(std::string\_view nodeName, std::string\_view eventType, const std::vector&lt; [**NodeEventKeyValue**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-nodeeventkeyvalue) &gt; &kvPairs)&gt; | [**NodeEventCallback**](#typedef-nodeeventcallback)  <br>_Generic_ `NodeEvent` _callback._ |
| typedef std::pair&lt; std::string, std::string &gt; | [**NodeEventKeyValue**](#typedef-nodeeventkeyvalue)  <br>_One ordered key/value pair in a_ `NodeEvent` _payload._ |
| typedef std::function&lt; void(std::string\_view nodeName, std::string\_view pendingExit)&gt; | [**PausedCallback**](#typedef-pausedcallback)  <br>_Callback type for pause notifications._  |
| typedef std::function&lt; void(std::string\_view toolName, std::string\_view argumentsJson)&gt; | [**ToolCallCallback**](#typedef-toolcallcallback)  <br>_Callback type for tool-call notifications._  |
| typedef std::function&lt; void(const [**ToolCallEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_tool_call_event.md) &event)&gt; | [**ToolCallEventCallback**](#typedef-toolcalleventcallback)  <br>_Preferred tool-call callback with owning payload._  |
| typedef std::function&lt; void(std::string\_view callId, std::string\_view toolName, std::string\_view argumentsJson)&gt; | [**ToolCallWithIdCallback**](#typedef-toolcallwithidcallback)  <br>_Call-ID-aware tool-call callback._  |
| typedef std::function&lt; void(std::string\_view nodeName, std::span&lt; const int16\_t &gt; pcm)&gt; | [**TtsAudioCallback**](#typedef-ttsaudiocallback)  <br>_Callback type for streaming TTS PCM chunks._  |
| typedef std::function&lt; void(std::string\_view nodeName, uint32\_t sampleRate, uint16\_t channels, uint16\_t bitsPerSample)&gt; | [**TtsAudioFormatCallback**](#typedef-ttsaudioformatcallback)  <br>_Callback type for the TTS audio format announcement._  |
| typedef std::function&lt; void(std::string\_view nodeName, std::span&lt; const [**VisemeEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_viseme_event.md) &gt; events)&gt; | [**TtsVisemesCallback**](#typedef-ttsvisemescallback)  <br>_Callback type for streaming TTS viseme frames._  |
| typedef std::function&lt; void(::Tryll::TurnStatus status, std::string\_view debugInfoJson, std::int32\_t tokensGenerated)&gt; | [**TurnCompleteCallback**](#typedef-turncompletecallback)  <br>_Callback type for turn-complete notifications._  |




















## Public Functions

| Type | Name |
| ---: | :--- |
|   | [**AgentProxy**](#function-agentproxy-12) () = default<br> |
|  std::uint32\_t | [**AppendInteractions**](#function-appendinteractions) (std::span&lt; const [**DialogInteraction**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_dialog_interaction.md) &gt; interactions) <br>_Blocking convenience wrapper over_ [_**AppendInteractionsAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-appendinteractionsasync) _._ |
|  std::future&lt; std::uint32\_t &gt; | [**AppendInteractionsAsync**](#function-appendinteractionsasync) (std::span&lt; const [**DialogInteraction**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_dialog_interaction.md) &gt; interactions) <br> |
|  void | [**Cancel**](#function-cancel) (::Tryll::CancelMode mode=::Tryll::CancelMode\_StopAndKeep) <br>_Cooperatively cancel the in-flight turn (fire-and-forget)._  |
|  void | [**ChangeParams**](#function-changeparams) (std::string\_view nodeName, [**NodeParamsVariant**](namespace_tryll_1_1_client.md#typedef-nodeparamsvariant) params) <br>_Blocking convenience wrapper over_ [_**ChangeParamsAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-changeparamsasync) _._ |
|  std::future&lt; void &gt; | [**ChangeParamsAsync**](#function-changeparamsasync) (std::string\_view nodeName, [**NodeParamsVariant**](namespace_tryll_1_1_client.md#typedef-nodeparamsvariant) params) <br>_Apply typed node parameters to a named workflow node asynchronously._  |
|  void | [**Destroy**](#function-destroy) () <br>_Blocking convenience wrapper over_ [_**DestroyAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-destroyasync) _._ |
|  std::future&lt; void &gt; | [**DestroyAsync**](#function-destroyasync) () <br>_Request agent destruction on the server asynchronously._  |
|  [**AgentKvCacheEvictResult**](struct_tryll_1_1_client_1_1_agent_kv_cache_evict_result.md) | [**EvictKvCache**](#function-evictkvcache) () <br>_Blocking convenience wrapper over_ [_**EvictKvCacheAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-evictkvcacheasync) _._ |
|  std::future&lt; [**AgentKvCacheEvictResult**](struct_tryll_1_1_client_1_1_agent_kv_cache_evict_result.md) &gt; | [**EvictKvCacheAsync**](#function-evictkvcacheasync) () <br>_Release raw eligible language-model contexts while retaining agent state._  |
|  [**DialogSnapshot**](struct_tryll_1_1_client_1_1_dialog_snapshot.md) | [**ExportDialog**](#function-exportdialog) () <br> |
|  std::future&lt; [**DialogSnapshot**](struct_tryll_1_1_client_1_1_dialog_snapshot.md) &gt; | [**ExportDialogAsync**](#function-exportdialogasync) () <br>_Export the agent's completed dialog as an owned snapshot. Idle-only._  |
|  std::uint64\_t | [**GetAgentId**](#function-getagentid) () noexcept const<br>_Server-assigned agent identifier._  |
|  [**AgentKvCacheStatus**](struct_tryll_1_1_client_1_1_agent_kv_cache_status.md) | [**GetKvCacheStatus**](#function-getkvcachestatus) () <br>_Blocking convenience wrapper over_ [_**GetKvCacheStatusAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-getkvcachestatusasync) _._ |
|  std::future&lt; [**AgentKvCacheStatus**](struct_tryll_1_1_client_1_1_agent_kv_cache_status.md) &gt; | [**GetKvCacheStatusAsync**](#function-getkvcachestatusasync) () <br>_Return aggregate KV-cache lifecycle state for this agent._  |
|  std::string | [**GetPausedNode**](#function-getpausednode) () const<br>_Node this agent's turn is currently paused after._  |
|  [**DialogImportResult**](struct_tryll_1_1_client_1_1_dialog_import_result.md) | [**ImportDialog**](#function-importdialog) (const [**DialogSnapshot**](struct_tryll_1_1_client_1_1_dialog_snapshot.md) & snapshot, DialogCompatibility compatibility=DialogCompatibility::Strict) <br> |
|  std::future&lt; [**DialogImportResult**](struct_tryll_1_1_client_1_1_dialog_import_result.md) &gt; | [**ImportDialogAsync**](#function-importdialogasync) (const [**DialogSnapshot**](struct_tryll_1_1_client_1_1_dialog_snapshot.md) & snapshot, DialogCompatibility compatibility=DialogCompatibility::Strict) <br>_Replace the agent's dialog with_ `snapshot` _. Idle-only. Never appends._ |
|  [**DialogFileLoadResult**](struct_tryll_1_1_client_1_1_dialog_file_load_result.md) | [**LoadDialog**](#function-loaddialog) (std::string\_view path, DialogCompatibility compatibility=DialogCompatibility::Strict, std::chrono::milliseconds timeout=std::chrono::seconds{30}) <br> |
|  std::future&lt; [**DialogFileLoadResult**](struct_tryll_1_1_client_1_1_dialog_file_load_result.md) &gt; | [**LoadDialogAsync**](#function-loaddialogasync) (std::string\_view path, DialogCompatibility compatibility=DialogCompatibility::Strict) <br> |
|  [**AgentKvCachePrefillResult**](struct_tryll_1_1_client_1_1_agent_kv_cache_prefill_result.md) | [**PrefillKvCache**](#function-prefillkvcache) () <br>_Blocking convenience wrapper over_ [_**PrefillKvCacheAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-prefillkvcacheasync) _._ |
|  std::future&lt; [**AgentKvCachePrefillResult**](struct_tryll_1_1_client_1_1_agent_kv_cache_prefill_result.md) &gt; | [**PrefillKvCacheAsync**](#function-prefillkvcacheasync) () <br>_Synchronise eligible language-model contexts to the reusable prefix._  |
|  std::uint32\_t | [**RemoveInteractionsFromEnd**](#function-removeinteractionsfromend) (std::uint32\_t count) <br>_Blocking convenience wrapper over_ [_**RemoveInteractionsFromEndAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-removeinteractionsfromendasync) _._ |
|  std::future&lt; std::uint32\_t &gt; | [**RemoveInteractionsFromEndAsync**](#function-removeinteractionsfromendasync) (std::uint32\_t count) <br> |
|  void | [**Resume**](#function-resume) (std::string\_view resumeNode={}) <br>_Blocking convenience wrapper over_ [_**ResumeAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-resumeasync) _._ |
|  std::future&lt; void &gt; | [**ResumeAsync**](#function-resumeasync) (std::string\_view resumeNode={}) <br>_Resume a turn that is currently paused (see_ [_**SetOnPaused**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-setonpaused) _)._ |
|  void | [**ResumeWithToolResult**](#function-resumewithtoolresult) (const [**ToolResult**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_tool_result.md) & toolResult, std::string\_view resumeNode={}) <br>_Blocking convenience wrapper over_ [_**ResumeWithToolResultAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-resumewithtoolresultasync) _._ |
|  std::future&lt; void &gt; | [**ResumeWithToolResultAsync**](#function-resumewithtoolresultasync) (const [**ToolResult**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_tool_result.md) & toolResult, std::string\_view resumeNode={}) <br>_Resume with one correlated tool result._  |
|  void | [**ResumeWithToolResults**](#function-resumewithtoolresults) (std::span&lt; const [**ToolResult**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_tool_result.md) &gt; toolResults, std::string\_view resumeNode={}) <br>_Blocking convenience wrapper over_ [_**ResumeWithToolResultsAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-resumewithtoolresultsasync) _._ |
|  std::future&lt; void &gt; | [**ResumeWithToolResultsAsync**](#function-resumewithtoolresultsasync) (std::span&lt; const [**ToolResult**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_tool_result.md) &gt; toolResults, std::string\_view resumeNode={}) <br>_Resume with the complete result batch for one paused tool-call pass._  |
|  [**DialogFileSaveResult**](struct_tryll_1_1_client_1_1_dialog_file_save_result.md) | [**SaveDialog**](#function-savedialog) (std::string\_view path, std::string\_view saveId, std::chrono::milliseconds timeout=std::chrono::seconds{30}) <br> |
|  std::future&lt; [**DialogFileSaveResult**](struct_tryll_1_1_client_1_1_dialog_file_save_result.md) &gt; | [**SaveDialogAsync**](#function-savedialogasync) (std::string\_view path, std::string\_view saveId) <br> |
|  void | [**SendText**](#function-sendtext) (std::string\_view text) <br>_Send a text message to the agent (fire-and-forget)._  |
|  void | [**SetOnAnswerText**](#function-setonanswertext) ([**AnswerTextCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-answertextcallback) cb) <br>_Register a callback for streaming text chunks._  |
|  void | [**SetOnError**](#function-setonerror) ([**ErrorCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-errorcallback) cb) <br>_Register a callback for agent-level error notifications._  |
|  void | [**SetOnIntentClassified**](#function-setonintentclassified) ([**IntentClassifiedCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-intentclassifiedcallback) cb) <br>_Register a callback for intent-classification notifications._  |
|  void | [**SetOnIntentLlmClassified**](#function-setonintentllmclassified) ([**IntentLlmClassifiedCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-intentllmclassifiedcallback) cb) <br>_Register a callback for LLM intent-classification notifications._  |
|  void | [**SetOnNodeEvent**](#function-setonnodeevent) ([**NodeEventCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-nodeeventcallback) cb) <br>_Register a generic_ `NodeEvent` _fallback callback._ |
|  void | [**SetOnPaused**](#function-setonpaused) ([**PausedCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-pausedcallback) cb) <br>_Register a callback for pause notifications._  |
|  void | [**SetOnToolCall**](#function-setontoolcall) ([**ToolCallCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-toolcallcallback) cb) <br>_Register a callback for tool-call notifications._  |
|  void | [**SetOnToolCallEvent**](#function-setontoolcallevent) ([**ToolCallEventCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-toolcalleventcallback) cb) <br>_Register the preferred owning tool-call callback._  |
|  void | [**SetOnToolCallWithId**](#function-setontoolcallwithid) ([**ToolCallWithIdCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-toolcallwithidcallback) cb) <br>_Register a call-ID-aware tool-call callback._  |
|  void | [**SetOnTtsAudio**](#function-setonttsaudio) ([**TtsAudioCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-ttsaudiocallback) cb) <br>_Register (or clear) the TTS audio chunk callback._  |
|  void | [**SetOnTtsAudioFormat**](#function-setonttsaudioformat) ([**TtsAudioFormatCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-ttsaudioformatcallback) cb) <br>_Register (or clear) the TTS audio format callback._  |
|  void | [**SetOnTtsVisemes**](#function-setonttsvisemes) ([**TtsVisemesCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-ttsvisemescallback) cb) <br>_Register (or clear) the TTS viseme-frame callback._  |
|  void | [**SetOnTurnComplete**](#function-setonturncomplete) ([**TurnCompleteCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-turncompletecallback) cb) <br>_Register a callback for turn-complete notifications._  |
|  [**AgentVariables**](class_tryll_1_1_client_1_1_agent_variables.md) & | [**Variables**](#function-variables) () <br>_The agent's declared variables (mirror + deferred-flush batcher)._  |




























## Detailed Description


## Public Types Documentation




### typedef AnswerTextCallback 

_Callback type for streaming text chunks._ 
```C++
using Tryll::Client::AgentProxy::AnswerTextCallback = std::function<
       void(std::string_view nodeName, std::string_view text, bool isDelta, bool isFinal)>;
```



Invoked on the reader thread for each `AnswerText` frame received from the server. Must return quickly and must not call any blocking [**TryllClient**](class_tryll_1_1_client_1_1_tryll_client.md) or [**AgentProxy**](class_tryll_1_1_client_1_1_agent_proxy.md) methods.


Parameters:
* `nodeName` — name of the workflow node that produced this text (multi-sender attribution; e.g. distinguishing two Generate nodes voicing different characters in the same turn).
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

_Callback type for embedding intent-classification notifications._ 
```C++
using Tryll::Client::AgentProxy::IntentClassifiedCallback = 
std::function<void(const IntentClassifiedEvent& event)>;
```



Invoked on the reader thread for each `NodeEvent` with `event_type="intent_classified"`. Must return quickly. 


        

<hr>



### typedef IntentLlmClassifiedCallback 

_Callback type for LLM intent-classification notifications._ 
```C++
using Tryll::Client::AgentProxy::IntentLlmClassifiedCallback = 
std::function<void(const IntentLlmClassifiedEvent& event)>;
```



Invoked on the reader thread for each `NodeEvent` with `event_type="intent_llm_classified"`. Must return quickly. 


        

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



### typedef PausedCallback 

_Callback type for pause notifications._ 
```C++
using Tryll::Client::AgentProxy::PausedCallback = 
std::function<void(std::string_view nodeName, std::string_view pendingExit)>;
```



Invoked on the reader thread for each `NodeEvent` with `event_type="paused"` — fired by the executor when it pauses the turn between nodes (see `PauseNode`, pausing `ToolCallDisposition` values).


Parameters:
* `nodeName` — the node that just exited and triggered the pause.
* `pendingExit` — the exit route that will be taken on a plain (non-jump) [**Resume**](class_tryll_1_1_client_1_1_agent_proxy.md#function-resume).




Must return quickly and must not call any blocking methods. 


        

<hr>



### typedef ToolCallCallback 

_Callback type for tool-call notifications._ 
```C++
using Tryll::Client::AgentProxy::ToolCallCallback = 
std::function<void(std::string_view toolName, std::string_view argumentsJson)>;
```



Invoked on the reader thread for each `NodeEvent` with `event_type="tool_call"` received from the server (i.e. when the agent's graph has a `ToolCall` node with a notifying disposition).


Parameters:
* `toolName` — the tool the model asked to call.
* `argumentsJson` — compact JSON object of arguments, e.g. `{"city"`:"Berlin"}.




Must return quickly and must not call any blocking methods. 


        

<hr>



### typedef ToolCallEventCallback 

_Preferred tool-call callback with owning payload._ 
```C++
using Tryll::Client::AgentProxy::ToolCallEventCallback = std::function<void(const ToolCallEvent& event)>;
```



Invoked on the reader thread. Must return quickly and must not call blocking client methods; use [**ResumeWithToolResultsAsync**](class_tryll_1_1_client_1_1_agent_proxy.md#function-resumewithtoolresultsasync) instead. 


        

<hr>



### typedef ToolCallWithIdCallback 

_Call-ID-aware tool-call callback._ 
```C++
using Tryll::Client::AgentProxy::ToolCallWithIdCallback = 
std::function<void(std::string_view callId,
                   std::string_view toolName,
                   std::string_view argumentsJson)>;
```



This is the result-bearing counterpart to [**ToolCallCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-toolcallcallback). The `callId` must be echoed in a [**ToolResult**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_tool_result.md) passed to [**ResumeWithToolResult**](class_tryll_1_1_client_1_1_agent_proxy.md#function-resumewithtoolresult) or [**ResumeWithToolResults**](class_tryll_1_1_client_1_1_agent_proxy.md#function-resumewithtoolresults). 


        

<hr>



### typedef TtsAudioCallback 

_Callback type for streaming TTS PCM chunks._ 
```C++
using Tryll::Client::AgentProxy::TtsAudioCallback = 
std::function<void(std::string_view nodeName, std::span<const int16_t> pcm)>;
```



Fired for each int16 LE PCM chunk synthesized by a GenerateAndSpeak/Speak node. End-of-stream is signalled by the existing turn-complete future returned from SendMessage — there is no per-chunk final flag.


Parameters:
* `nodeName` — name of the producing node (multi-sender attribution).
* `pcm` — the PCM chunk.




Must return quickly and must not call any blocking methods. 


        

<hr>



### typedef TtsAudioFormatCallback 

_Callback type for the TTS audio format announcement._ 
```C++
using Tryll::Client::AgentProxy::TtsAudioFormatCallback = 
std::function<void(std::string_view nodeName, uint32_t sampleRate, uint16_t channels, uint16_t bitsPerSample)>;
```



Fired once per GenerateAndSpeak/Speak turn, before the first TtsAudioFrame.


Parameters:
* `nodeName` — name of the producing node (multi-sender attribution).
* `sampleRate` — PCM sample rate (e.g. 22050).
* `channels` — channel count (always 1 for current models).
* `bitsPerSample` — bit depth (always 16). 




        

<hr>



### typedef TtsVisemesCallback 

_Callback type for streaming TTS viseme frames._ 
```C++
using Tryll::Client::AgentProxy::TtsVisemesCallback = 
std::function<void(std::string_view nodeName, std::span<const VisemeEvent> events)>;
```



Fired after each synthesized sentence when the producing GenerateAndSpeak node has `emit_visemes` set. The frame trails that sentence's audio frames (timing needs the final sample count). The contract is that events for a sample range arrive before that range plays under normal client buffering.


Parameters:
* `nodeName` — name of the producing node (multi-sender attribution).
* `events` — the viseme events in this frame (span valid only for the duration of the callback).




Must return quickly and must not call any blocking methods. Same threading as [**SetOnTtsAudio**](class_tryll_1_1_client_1_1_agent_proxy.md#function-setonttsaudio) (dispatch thread). 


        

<hr>



### typedef TurnCompleteCallback 

_Callback type for turn-complete notifications._ 
```C++
using Tryll::Client::AgentProxy::TurnCompleteCallback = 
std::function<void(::Tryll::TurnStatus status,
                   std::string_view debugInfoJson,
                   std::int32_t tokensGenerated)>;
```



Invoked on the reader thread once per turn, immediately after `TurnComplete` arrives. Covers client-initiated turns (via [**SendText**](class_tryll_1_1_client_1_1_agent_proxy.md#function-sendtext)).


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



### function AppendInteractions 

_Blocking convenience wrapper over_ [_**AppendInteractionsAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-appendinteractionsasync) _._
```C++
std::uint32_t Tryll::Client::AgentProxy::AppendInteractions (
    std::span< const DialogInteraction > interactions
) 
```




<hr>



### function AppendInteractionsAsync 

```C++
std::future< std::uint32_t > Tryll::Client::AgentProxy::AppendInteractionsAsync (
    std::span< const DialogInteraction > interactions
) 
```



Append scripted interactions without running the graph. Idle-only. 

**Returns:**

Number of interactions actually appended. 





        

<hr>



### function Cancel 

_Cooperatively cancel the in-flight turn (fire-and-forget)._ 
```C++
void Tryll::Client::AgentProxy::Cancel (
    ::Tryll::CancelMode mode=::Tryll::CancelMode_StopAndKeep
) 
```



Stops generation; the active [**SendText**](class_tryll_1_1_client_1_1_agent_proxy.md#function-sendtext) turn ends with `TurnStatus::Cancelled` through [**SetOnTurnComplete**](class_tryll_1_1_client_1_1_agent_proxy.md#function-setonturncomplete). No-op if the agent is idle. `mode` selects whether the partial interaction is kept (default) or discarded as if the turn never happened.




**Parameters:**


* `mode` Cancel disposition (StopAndKeep / StopAndDiscard). 




        

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



### function EvictKvCache 

_Blocking convenience wrapper over_ [_**EvictKvCacheAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-evictkvcacheasync) _._
```C++
AgentKvCacheEvictResult Tryll::Client::AgentProxy::EvictKvCache () 
```




<hr>



### function EvictKvCacheAsync 

_Release raw eligible language-model contexts while retaining agent state._ 
```C++
std::future< AgentKvCacheEvictResult > Tryll::Client::AgentProxy::EvictKvCacheAsync () 
```




<hr>



### function ExportDialog 

```C++
DialogSnapshot Tryll::Client::AgentProxy::ExportDialog () 
```




<hr>



### function ExportDialogAsync 

_Export the agent's completed dialog as an owned snapshot. Idle-only._ 
```C++
std::future< DialogSnapshot > Tryll::Client::AgentProxy::ExportDialogAsync () 
```




<hr>



### function GetAgentId 

_Server-assigned agent identifier._ 
```C++
inline std::uint64_t Tryll::Client::AgentProxy::GetAgentId () noexcept const
```





**Returns:**

The `agent_id` field from the original `CreateAgentResponse`. 





        

<hr>



### function GetKvCacheStatus 

_Blocking convenience wrapper over_ [_**GetKvCacheStatusAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-getkvcachestatusasync) _._
```C++
AgentKvCacheStatus Tryll::Client::AgentProxy::GetKvCacheStatus () 
```




<hr>



### function GetKvCacheStatusAsync 

_Return aggregate KV-cache lifecycle state for this agent._ 
```C++
std::future< AgentKvCacheStatus > Tryll::Client::AgentProxy::GetKvCacheStatusAsync () 
```




<hr>



### function GetPausedNode 

_Node this agent's turn is currently paused after._ 
```C++
std::string Tryll::Client::AgentProxy::GetPausedNode () const
```



Empty when the turn is not paused. A non-empty value means the server is holding the turn open until [**Resume**](class_tryll_1_1_client_1_1_agent_proxy.md#function-resume) (or [**Cancel**](class_tryll_1_1_client_1_1_agent_proxy.md#function-cancel)) — until then SendMessage keeps failing with `AgentBusy` (3004).


Thread-safe; reflects the most recent `paused` / `TurnComplete` frame. 


        

<hr>



### function ImportDialog 

```C++
DialogImportResult Tryll::Client::AgentProxy::ImportDialog (
    const DialogSnapshot & snapshot,
    DialogCompatibility compatibility=DialogCompatibility::Strict
) 
```




<hr>



### function ImportDialogAsync 

_Replace the agent's dialog with_ `snapshot` _. Idle-only. Never appends._
```C++
std::future< DialogImportResult > Tryll::Client::AgentProxy::ImportDialogAsync (
    const DialogSnapshot & snapshot,
    DialogCompatibility compatibility=DialogCompatibility::Strict
) 
```




<hr>



### function LoadDialog 

```C++
DialogFileLoadResult Tryll::Client::AgentProxy::LoadDialog (
    std::string_view path,
    DialogCompatibility compatibility=DialogCompatibility::Strict,
    std::chrono::milliseconds timeout=std::chrono::seconds{30}
) 
```




<hr>



### function LoadDialogAsync 

```C++
std::future< DialogFileLoadResult > Tryll::Client::AgentProxy::LoadDialogAsync (
    std::string_view path,
    DialogCompatibility compatibility=DialogCompatibility::Strict
) 
```




<hr>



### function PrefillKvCache 

_Blocking convenience wrapper over_ [_**PrefillKvCacheAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-prefillkvcacheasync) _._
```C++
AgentKvCachePrefillResult Tryll::Client::AgentProxy::PrefillKvCache () 
```




<hr>



### function PrefillKvCacheAsync 

_Synchronise eligible language-model contexts to the reusable prefix._ 
```C++
std::future< AgentKvCachePrefillResult > Tryll::Client::AgentProxy::PrefillKvCacheAsync () 
```




<hr>



### function RemoveInteractionsFromEnd 

_Blocking convenience wrapper over_ [_**RemoveInteractionsFromEndAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-removeinteractionsfromendasync) _._
```C++
std::uint32_t Tryll::Client::AgentProxy::RemoveInteractionsFromEnd (
    std::uint32_t count
) 
```




<hr>



### function RemoveInteractionsFromEndAsync 

```C++
std::future< std::uint32_t > Tryll::Client::AgentProxy::RemoveInteractionsFromEndAsync (
    std::uint32_t count
) 
```



Remove whole interactions from the end of the dialog. Idle-only. count=0 no-op; count&gt;=size clears all. 

**Returns:**

Number actually removed. 





        

<hr>



### function Resume 

_Blocking convenience wrapper over_ [_**ResumeAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-resumeasync) _._
```C++
void Tryll::Client::AgentProxy::Resume (
    std::string_view resumeNode={}
) 
```





**Exception:**


* `TryllError` On server-reported errors or disconnect. 




        

<hr>



### function ResumeAsync 

_Resume a turn that is currently paused (see_ [_**SetOnPaused**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-setonpaused) _)._
```C++
std::future< void > Tryll::Client::AgentProxy::ResumeAsync (
    std::string_view resumeNode={}
) 
```





**Parameters:**


* `resumeNode` Empty continues via the paused node's pending exit route; non-empty jumps to that node by name.



**Returns:**

Future completing on `Ack`. Throws [**TryllError**](class_tryll_1_1_client_1_1_tryll_error.md) on server-reported errors (`AgentNotPaused` 3012 if the agent is not currently paused, `UnknownNode` 3005 if `resumeNode` does not name a node in the graph). 





        

<hr>



### function ResumeWithToolResult 

_Blocking convenience wrapper over_ [_**ResumeWithToolResultAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-resumewithtoolresultasync) _._
```C++
void Tryll::Client::AgentProxy::ResumeWithToolResult (
    const ToolResult & toolResult,
    std::string_view resumeNode={}
) 
```




<hr>



### function ResumeWithToolResultAsync 

_Resume with one correlated tool result._ 
```C++
std::future< void > Tryll::Client::AgentProxy::ResumeWithToolResultAsync (
    const ToolResult & toolResult,
    std::string_view resumeNode={}
) 
```




<hr>



### function ResumeWithToolResults 

_Blocking convenience wrapper over_ [_**ResumeWithToolResultsAsync**_](class_tryll_1_1_client_1_1_agent_proxy.md#function-resumewithtoolresultsasync) _._
```C++
void Tryll::Client::AgentProxy::ResumeWithToolResults (
    std::span< const ToolResult > toolResults,
    std::string_view resumeNode={}
) 
```




<hr>



### function ResumeWithToolResultsAsync 

_Resume with the complete result batch for one paused tool-call pass._ 
```C++
std::future< void > Tryll::Client::AgentProxy::ResumeWithToolResultsAsync (
    std::span< const ToolResult > toolResults,
    std::string_view resumeNode={}
) 
```




<hr>



### function SaveDialog 

```C++
DialogFileSaveResult Tryll::Client::AgentProxy::SaveDialog (
    std::string_view path,
    std::string_view saveId,
    std::chrono::milliseconds timeout=std::chrono::seconds{30}
) 
```




<hr>



### function SaveDialogAsync 

```C++
std::future< DialogFileSaveResult > Tryll::Client::AgentProxy::SaveDialogAsync (
    std::string_view path,
    std::string_view saveId
) 
```




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


Fires on the reader thread for every `AnswerText` frame.




**Parameters:**


* `cb` Callable invoked with ``(nodeName, text, isDelta, isFinal), or empty to clear. 




        

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


Fires on the reader thread for every `NodeEvent` with `event_type="intent_classified"` (embedding ClassifyIntent).




**Parameters:**


* `cb` Callable invoked with an [**IntentClassifiedEvent**](struct_tryll_1_1_client_1_1_agent_proxy_1_1_intent_classified_event.md), or empty to clear. 




        

<hr>



### function SetOnIntentLlmClassified 

_Register a callback for LLM intent-classification notifications._ 
```C++
void Tryll::Client::AgentProxy::SetOnIntentLlmClassified (
    IntentLlmClassifiedCallback cb
) 
```



Replaces any previously set callback. Pass an empty (default- constructed) [**IntentLlmClassifiedCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-intentllmclassifiedcallback) to unregister.


Fires on the reader thread for every `NodeEvent` with `event_type="intent_llm_classified"`. 


        

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



### function SetOnPaused 

_Register a callback for pause notifications._ 
```C++
void Tryll::Client::AgentProxy::SetOnPaused (
    PausedCallback cb
) 
```



Replaces any previously set callback. Pass an empty (default- constructed) [**PausedCallback**](class_tryll_1_1_client_1_1_agent_proxy.md#typedef-pausedcallback) to unregister.


Fires on the reader thread for every `NodeEvent` with `event_type="paused"` the server sends for this agent.




**Parameters:**


* `cb` Callable invoked with ``(nodeName, pendingExit), or empty to clear. 




        

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



### function SetOnToolCallEvent 

_Register the preferred owning tool-call callback._ 
```C++
void Tryll::Client::AgentProxy::SetOnToolCallEvent (
    ToolCallEventCallback cb
) 
```



Invoked in addition to any legacy [**SetOnToolCall**](class_tryll_1_1_client_1_1_agent_proxy.md#function-setontoolcall) / [**SetOnToolCallWithId**](class_tryll_1_1_client_1_1_agent_proxy.md#function-setontoolcallwithid) callbacks. Pass an empty callback to unregister.




**Warning:**

The legacy `string_view` callbacks receive views into the inbound frame buffer that are valid only for the duration of the synchronous callback. Use this API when capturing fields for deferred work. 





        

<hr>



### function SetOnToolCallWithId 

_Register a call-ID-aware tool-call callback._ 
```C++
void Tryll::Client::AgentProxy::SetOnToolCallWithId (
    ToolCallWithIdCallback cb
) 
```



The legacy [**SetOnToolCall**](class_tryll_1_1_client_1_1_agent_proxy.md#function-setontoolcall) callback remains supported and is also invoked when registered. Pass an empty callback to unregister. 


        

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



### function SetOnTtsVisemes 

_Register (or clear) the TTS viseme-frame callback._ 
```C++
void Tryll::Client::AgentProxy::SetOnTtsVisemes (
    TtsVisemesCallback cb
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




        

<hr>



### function Variables 

_The agent's declared variables (mirror + deferred-flush batcher)._ 
```C++
AgentVariables & Tryll::Client::AgentProxy::Variables () 
```



See [**Tryll::Client::AgentVariables**](class_tryll_1_1_client_1_1_agent_variables.md). Writes staged here are flushed automatically as one batched request immediately before the next [**SendText**](class_tryll_1_1_client_1_1_agent_proxy.md#function-sendtext) / [**Resume**](class_tryll_1_1_client_1_1_agent_proxy.md#function-resume) / [**ResumeAsync**](class_tryll_1_1_client_1_1_agent_proxy.md#function-resumeasync) / [**ChangeParams**](class_tryll_1_1_client_1_1_agent_proxy.md#function-changeparams) / [**ChangeParamsAsync**](class_tryll_1_1_client_1_1_agent_proxy.md#function-changeparamsasync) call (not [**Cancel**](class_tryll_1_1_client_1_1_agent_proxy.md#function-cancel)) — see docs/research/workflow/per-agent-variables-requirements.md §6.3.




**Returns:**

Reference to the mirror, valid for the lifetime of the underlying client connection. 





        

<hr>## Friends Documentation





### friend TryllClientImpl 

```C++
class Tryll::Client::AgentProxy::TryllClientImpl (
    Internal::TryllClientImpl
) 
```




<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo3/tryll/clients/cpp/include/tryll/AgentProxy.h`

