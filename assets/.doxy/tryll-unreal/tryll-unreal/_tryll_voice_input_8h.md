

# File TryllVoiceInput.h



[**FileList**](files.md) **>** [**clients**](dir_ae1e47b40792601544f85532b4958859.md) **>** [**unreal**](dir_b8761365e93ebda0e5697455672eef41.md) **>** [**Source**](dir_2d515141515bf2e74d881ba79f9137e4.md) **>** [**TryllClient**](dir_86e4d1eacb47bf47a46b7a1cbe7617d1.md) **>** [**Public**](dir_ce990ac36c6f0b3bdd019ac68edf26a8.md) **>** [**TryllVoiceInput.h**](_tryll_voice_input_8h.md)





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
The documentation for this class was generated from the following file `C:/_tryll/_monorepo2/tryll/clients/unreal/Source/TryllClient/Public/TryllVoiceInput.h`

