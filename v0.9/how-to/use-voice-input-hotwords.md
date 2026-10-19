# Bias Voice Input with Hotwords

Open-vocabulary speech-to-text engines guess every word from acoustic evidence alone, which means they often mangle game-specific proper nouns — the NPC name "Aeltharion" becomes "Al Therion", the spell "frostbolt" becomes "frost bolt". Sherpa-ONNX exposes a *hotwords* list that nudges the decoder toward a designer-authored set of phrases without retraining the model.

This recipe shows how to ship a phrase list with your game and apply it to a `VoiceInput`.

## Prerequisites

- A working `VoiceInput` session — see [Use Voice Input](https://docs.tryllengine.com/v0.9/how-to/use-voice-input.html) for the full setup walkthrough.
- An STT model that supports hotwords. Most transducer / CTC families do; encoder-decoder families (Whisper, SenseVoice, Moonshine, Canary) log a warning and ignore the list when the voice input is created. The matrix and the warning are in [Which models support hotwords](#which-models-support-hotwords) below.

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

The file path is **relative to the process storage root** (Project Settings **Storage Data Folder**, default `TryllStorage/`). The server loads it on demand the first time a `VoiceInput` references it — no `CreateStringStorage` call is needed.

## 2. Reference the file when creating `VoiceInput`

**C++**

```
#include <tryll/TryllClient.h>
#include <tryll/VoiceInput.h>

Tryll::Client::VoiceInputConfig viCfg;
viCfg.modelName            = "Parakeet TDT 0.6B v3 (int8)";
viCfg.inputFormat          = micFormat;
viCfg.hotwordsStoragePath  = "data/voice-hotwords.txt";
viCfg.hotwordsScore        = 1.8f;   // bonus per matched phrase; default 1.5

auto vi = client.CreateVoiceInput(viCfg);
```

**Python**

```
voice = client.create_voice_input(
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
    VadMinSilenceMs      = 1000,
    VadSpeechPadMs       = 250,
    HotwordsStoragePath  = "data/voice-hotwords.txt",
    HotwordsScore        = 1.8f,
};
var (voice, err) = await client.CreateVoiceInputAsync(cfg);
```

On `TryllVoiceInputComponent`, set the same values in the inspector: `Hotwords Storage Path` (relative to Storage Data Folder; empty disables) and `Hotwords Score` (default 1.5).

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

The referenced file must exist under the process storage root; a missing file fails `CreateVoiceInput` with `StorageFileNotFound`.

## Picking a score

The score is the bonus added to a matched phrase during beam search (default 1.5). No score value turns biasing off: an empty `hotwordsStoragePath` is what disables it.

| Score     | Bias             | Use when…                                                                                         |
| --------- | ---------------- | ------------------------------------------------------------------------------------------------- |
| 1.0       | light            | a smaller bonus than the default; still biases                                                    |
| 1.5       | gentle (default) | the lexicon is broad and you do not want false positives                                          |
| 1.8 – 2.5 | moderate         | proper nouns + spell names — typical game lexicon                                                 |
| 3.0 – 4.0 | aggressive       | the recognizer keeps picking a phonetically-similar real word and you accept some false positives |

The score applies to every phrase in the list — unless a line sets its own. Append `:<number>` to a lexicon line to bias that phrase alone:

```
Kaelthorne :3.0
Veyra
frostbolt
```

Here `Kaelthorne` is biased at 3.0 and the rest at whatever the session's `hotwordsScore` is. Use it for the one name a recognizer keeps mangling, instead of raising the score for the whole list and paying for it everywhere.

### The score you set is one of two

The server applies hotword bias in two places, and only one of them is yours to set per session:

|                     | What it does                                                | Where it is set                                              |
| ------------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| **Candidate boost** | makes a hotword's tokens reachable by the recognizer at all | the server's model catalog (`stt_hotwords_score`), per model |
| **Phrase weight**   | weights matched phrases against one another                 | `hotwordsScore` on the VoiceInput — the one in this guide    |

The catalog value is what lets the recognizer consider `Kaelthorne` in the first place; `hotwordsScore` then decides how strongly `Kaelthorne` competes with the other phrases in your lexicon. You normally only touch the second.

### Turning biasing on for a model

Biasing needs beam-search decoding, which the server chooses when it loads the model — so it is a per-model decision, not something a single voice input can switch on. The shipped catalog enables it for **Parakeet TDT 0.6B v3** and no other model.

To change that for your project, use the **Hotword biasing** control in the Model Manager (beside Memory placement) — it is available on STT models:

|             |                                                                                                                                                                                         |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Default** | follow the server's setting for this model                                                                                                                                              |
| **Off**     | never bias. Supplying a lexicon then fails with `HotwordBiasDisabled` (4106) instead of quietly doing nothing                                                                           |
| **On**      | always bias. Beam-search decoding is then in force for every voice input on this model, costing a few percent of decode time and changing its transcripts — *on but unused is not free* |

From code, the same choice is a per-model hint on the session: `hotword_hints` on `create_session` in Python; in C++, a `TryllClient::ModelPlacementHint` with `hotwords` set, passed in `SessionConfig::placementHints`:

```
using Tryll::Client::TryllClient;

TryllClient::SessionConfig cfg;
cfg.placementHints.push_back({
    .modelName = "Parakeet TDT 0.6B v3 (int8)",
    .hotwords  = ::Tryll::HotwordBias_On,
});
client.CreateSession(cfg);
```

```
from tryll_client import HotwordBias

client.create_session(hotword_hints={"Parakeet TDT 0.6B v3 (int8)": HotwordBias.On})
```

If hotwords seem to do nothing at all on a model — no value of `hotwordsScore` changes anything — that model is not enabled for biasing. Check the control above, and the server log, which says `hotwords=unavailable` on the model's load line.

**Raising the score is not free, and past a point it is actively destructive.** Bias nudges the beam toward your phrases everywhere, including in speech that contains none of them, and short utterances have the least context to resist it. Measured on Parakeet TDT 0.6B v3 with a nine-phrase fantasy lexicon, at score 6.0:

| Spoken                  | Transcribed                             |
| ----------------------- | --------------------------------------- |
| "Fireball"              | "Mord"                                  |
| "Hello, how are you?"   | "Mords soul. Mordrian souls souls soul" |
| a four-sentence passage | unaffected                              |

So tune against two sets, not one: utterances that contain your terms, and utterances that do not. The second set is what tells you when to stop. Start at 1.5, raise until the terms land, then check the second set before keeping the value.

## Which models support hotwords

Two things have to be true: the decoder must have a beam for the bias to act on, and the model bundle must carry a sub-word vocabulary the server can encode your phrases with.

| Family                                                           | Hotwords             |
| ---------------------------------------------------------------- | -------------------- |
| Zipformer transducer / Parakeet TDT (Transducer, NemoTransducer) | yes                  |
| Streaming Zipformer (online transducer)                          | yes                  |
| Zipformer-CTC, NeMo CTC, Paraformer, Dolphin, WeNet CTC          | no — no beam to bias |
| Whisper                                                          | **no**               |
| SenseVoice                                                       | **no**               |
| Moonshine, Canary, FireRedASR, CohereTranscribe                  | **no**               |

When you pass a hotwords list to a model that cannot use it, the server logs a warning when the voice input is created and ignores the phrases, rather than accepting the list silently:

```
[Sherpa-ONNX] Hotword biasing unavailable for STT family 'whisper'
              (this decoder has no token-level bias hook); ignoring
              9 supplied phrase(s) on this session.
```

The model's load banner also states it up front:

```
[Sherpa-ONNX] STT model loaded: … | family=nemo_transducer | … |
              decoding=modified_beam_search | hotwords=available
```

The session is otherwise unaffected. Hotword-capable models load with beam-search decoding, which costs a few percent of decode time.

## Caveats

- **Write plain words.** The server hands your phrases to the recognizer's own tokenizer, so `Aeltharion` is what you want in the file — not a pre-tokenized `▁Ael th ar ion`. Lexicons written in the older `▁`-piece form still work; the server folds them back to graphemes. There is no longer any reason to run `text2token.py` over a lexicon by hand.
- **A whole utterance biases better than a fragment.** The recognizer decodes one speech segment at a time, and a phrase near a segment boundary has less context to be recognised from. If a term is being missed, check whether the pause structure of the speech is splitting the sentence before blaming the score.
- **The score is the default for the whole voice input.** A lexicon line that ends with `:<number>` overrides it for that phrase only.
- **The lexicon is loaded from the process storage root.** The server reads the file on demand and caches it. Ship the file with your game data and every session on that server can use it.
- **Out-of-vocabulary characters are dropped.** If your lexicon contains characters the model's tokenizer cannot represent (e.g. CJK characters against a pure-English Zipformer), they are silently dropped from the bias.

## Related

- [Use Voice Input](https://docs.tryllengine.com/v0.9/how-to/use-voice-input.html)
- [`StringStorage` reference](https://docs.tryllengine.com/v0.9/reference/string-storage.html)
