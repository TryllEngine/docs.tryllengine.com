# Bias Voice Input with Hotwords

Open-vocabulary speech-to-text engines guess every word from acoustic evidence alone, which means they often mangle game-specific proper nouns — the NPC name "Aeltharion" becomes "Al Therion", the spell "frostbolt" becomes "frost bolt". Sherpa-ONNX exposes a *hotwords* list that nudges the decoder toward a designer-authored set of phrases without retraining the model.

This recipe shows how to ship a phrase list with your game and apply it to a `VoiceInput`.

## Prerequisites

- A working `VoiceInput` session — see [Use Voice Input](https://docs.tryllengine.com/how-to/use-voice-input.html) for the full setup walkthrough.
- An STT model that supports hotwords. Most transducer / CTC families do; encoder-decoder families (Whisper, SenseVoice, Moonshine, Canary) silently ignore the list. The matrix is in [Which models support hotwords](#which-models-support-hotwords) below.

## 1. Author a phrase list

Hotwords live in a `StringStorage` of kind `List`, shipped as a UTF-8 text file in your game's storage data folder — one phrase per line. Blank lines and lines starting with `#` are ignored:

```
# data/voice-hotwords.txt
# fantasy NPC names
Aeltharion
Cor'than
Bjornthor

# spells / abilities
fireball
frostbolt
magic missile
```

The file path is **relative to the session storage folder** (set via `storageDataFolder` / `StorageDataFolder` on the session config). The server loads it on demand the first time a `VoiceInput` references it — no `CreateStringStorage` call is needed.

## 2. Reference the file when creating `VoiceInput`

**C++**

```
#include <tryll/TryllClient.h>
#include <tryll/VoiceInput.h>

Tryll::Client::VoiceInputConfig viCfg;
viCfg.modelName            = "Parakeet TDT 0.6B v3 (int8)";
viCfg.inputFormat          = micFormat;
viCfg.hotwordsStoragePath  = "data/voice-hotwords.txt";
viCfg.hotwordsScore        = 1.8f;   // 1.0 = no bias, 2.5 = aggressive

auto vi = client.CreateVoiceInput(viCfg);
```

**Python**

```
voice_input_id = client.create_voice_input(
    model_name="Parakeet TDT 0.6B v3 (int8)",
    sample_rate=16000, channels=1, bits_per_sample=16,
    hotwords_storage_path="data/voice-hotwords.txt",
    hotwords_score=1.8,
)
```

**Unity (C#)**

```
var cfg = new VoiceInputConfig
{
    ModelName            = "Parakeet TDT 0.6B v3 (int8)",
    InputFormat          = AudioFormat.Default,
    VadThreshold         = 0.5f,
    VadMinSilenceMs      = 500,
    VadSpeechPadMs       = 250,
    HotwordsStoragePath  = "data/voice-hotwords.txt",
    HotwordsScore        = 1.8f,
};
var (voice, err) = await client.CreateVoiceInputAsync(cfg);
```

**Unreal (C++)**

```
FTryllVoiceInputConfig Config;
Config.ModelName             = TEXT("Parakeet TDT 0.6B v3 (int8)");
Config.InputFormat.SampleRate = 16000;
Config.HotwordsStoragePath   = TEXT("data/voice-hotwords.txt");
Config.HotwordsScore         = 1.8f;

Subsystem->CreateVoiceInput(Config,
    [](TSharedPtr<FTryllVoiceInput> Voice, FTryllError Err)
    {
        if (!Err.IsOk()) { /* handle error */ return; }
        // Voice is ready — call BeginUtterance to start capturing.
        // BeginUtterance options include auto_finish_on_silence and
        // max_utterance_ms.
    });
```

The referenced file must exist under the session storage folder; a missing file fails `CreateVoiceInput` with `StorageFileNotFound`.

## Picking a score

| Score     | Bias             | Use when…                                                                                         |
| --------- | ---------------- | ------------------------------------------------------------------------------------------------- |
| 1.0       | none / off       | sanity check that the path is wired but you do not yet want bias                                  |
| 1.5       | gentle (default) | the lexicon is broad and you do not want false positives                                          |
| 1.8 – 2.0 | moderate         | proper nouns + spell names — typical game lexicon                                                 |
| 2.5+      | aggressive       | the recognizer keeps picking a phonetically-similar real word and you accept some false positives |

The score is applied to every phrase in the list. A future revision may allow per-phrase overrides.

## Which models support hotwords

| Family                                                           | Hotwords                  |
| ---------------------------------------------------------------- | ------------------------- |
| Zipformer transducer / Parakeet TDT (Transducer, NemoTransducer) | yes                       |
| Zipformer-CTC, NeMo CTC, Paraformer, Dolphin, WeNet CTC          | yes                       |
| Omnilingual, MedASR, TeleSpeech                                  | yes                       |
| Whisper                                                          | **no — silently ignored** |
| SenseVoice                                                       | **no — silently ignored** |
| Moonshine, Canary, FireRedASR, CohereTranscribe                  | **no**                    |

When you pass a hotwords list to an unsupported model the server emits a `warn`-level log line at session creation:

```
[Sherpa-ONNX] STT family 'whisper' does not support hotwords;
              ignoring 9 supplied phrase(s) on this session.
```

The session is otherwise unaffected.

## Caveats

- **BPE encoding is approximate in v1.** The server takes your grapheme phrases and prefixes each whitespace-separated word with the BPE word marker (`▁`). This works perfectly for words that the model's BPE tokenizer represents as a single piece — i.e. most common English verbs and nouns. Multi-piece words (the made-up fantasy name "Aeltharion" decomposes into 3–4 pieces) get partial bias, not the full effect.
- **Pre-tokenize for full effect.** If a particular proper noun matters and the approximate encoding is not biasing strongly enough, run [sherpa-onnx's `text2token.py`](https://k2-fsa.github.io/sherpa/onnx/hotwords/index.html) with the model's tokenizer over your lexicon and put the `▁Ael th ar ion`-style form directly in the storage. The server detects a leading `▁` and passes the phrase through unchanged.
- **The score is recognizer-wide per session.** It is not per-phrase.
- **The lexicon is loaded from the session storage folder.** The server reads the file on demand and caches it for the session lifetime. Ship the file with your game data and it is available to every session that points at the same storage folder.
- **Out-of-vocabulary characters are dropped.** The naïve encoder treats characters one-to-one. If your lexicon contains characters the tokenizer cannot represent (e.g. CJK characters against a pure-English Zipformer), they will be silently dropped from the bias.

## Related

- [Use Voice Input](https://docs.tryllengine.com/how-to/use-voice-input.html)
- [`StringStorage` reference](https://docs.tryllengine.com/reference/string-storage.html)
