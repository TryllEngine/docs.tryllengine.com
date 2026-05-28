

# File TryllVoiceInput.h



[**FileList**](files.md) **>** [**client-unreal**](dir_95d666eee9112f2bfa7f2b736b6243b9.md) **>** [**Source**](dir_e37f8a870dc803113e92bf135247a735.md) **>** [**TryllClient**](dir_0869abba98a308e3c3eadd7e169e0f62.md) **>** [**Public**](dir_338741d27b4bda5805009de80ddaf6fc.md) **>** [**TryllVoiceInput.h**](_tryll_voice_input_8h.md)





* `#include "CoreMinimal.h"`
* `#include "TryllError.h"`
* `#include "TryllVoiceInput.generated.h"`















## Classes

| Type | Name |
| ---: | :--- |
| struct | [**FTryllAudioFormat**](struct_f_tryll_audio_format.md) <br> |
| struct | [**FTryllTranscriptUpdate**](struct_f_tryll_transcript_update.md) <br> |
| struct | [**FTryllUtteranceOptions**](struct_f_tryll_utterance_options.md) <br> |
| class | [**FTryllVoiceInput**](class_f_tryll_voice_input.md) <br> |
| struct | [**FTryllVoiceInputConfig**](struct_f_tryll_voice_input_config.md) <br> |


## Public Types

| Type | Name |
| ---: | :--- |
| enum uint8 | [**ETryllTranscriptUpdateKind**](#enum-etrylltranscriptupdatekind)  <br> |
















































## Public Types Documentation




### enum ETryllTranscriptUpdateKind 

```C++
enum ETryllTranscriptUpdateKind {
    SpeechStart = 0,
    Partial = 1,
    SegmentFinal = 2,
    UtteranceFinal = 3
};
```



Classifies a transcript update. Mirrors the wire schema enum Tryll::TranscriptUpdateKind — ordinal values must stay in sync. 


        

<hr>

------------------------------
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/server/client-unreal/Source/TryllClient/Public/TryllVoiceInput.h`

