# STT Models

Tryll recognizes speech with one engine, [`SherpaOnnx`](https://docs.tryllengine.com/v0.10/concepts/models-and-inference-engines.html). It runs on the CPU, so a recognizer costs system RAM but no VRAM.

This page lists **which speech recognizers the Tryll server ships**, how they differ, and what each one needs. For the mental model behind voice input, see [STT and Voice Input](https://docs.tryllengine.com/v0.10/concepts/stt-and-voice-input.html). For the API, see [Use Voice Input](https://docs.tryllengine.com/v0.10/how-to/use-voice-input.html).

Short version: every shipped recognizer is either **offline** or **streaming**. Offline recognizers need the **Silero VAD** model on disk to cut speech into segments. Streaming recognizers have their own endpointer. Only **Parakeet TDT 0.6B v3** biases toward hotwords out of the box.

Models are never downloaded automatically. Get each one, including `Silero VAD`, with the Model Manager or `DownloadModelRequest`. For the catalog file format, see [Model Management](https://docs.tryllengine.com/v0.10/reference/model-management.html).

______________________________________________________________________

## Shipped recognizers

The entries below are the `"audience": "public"` STT entries in the default `models.json`. Release builds keep only `public` entries, so these are the recognizers in the shipped catalog. Other STT entries in the source catalog are not shipped.

You pick a recognizer by its exact catalog name, in the voice input's `model_name` (`ModelName` in Unity and Unreal). The engine comes from the catalog. You do not choose it.

| Catalog name                                  | Engine        | Family              | Mode      | Languages                           | Hotwords               | Weights licence                         |
| --------------------------------------------- | ------------- | ------------------- | --------- | ----------------------------------- | ---------------------- | --------------------------------------- |
| `Whisper Tiny EN (int8)`                      | `sherpa-onnx` | `whisper`           | Offline   | English                             | No                     | MIT                                     |
| `Whisper Base EN (int8)`                      | `sherpa-onnx` | `whisper`           | Offline   | English                             | No                     | MIT                                     |
| `Whisper Base Multilingual (int8)`            | `sherpa-onnx` | `whisper`           | Offline   | [42](#whisper-multilingual)         | No                     | MIT                                     |
| `Whisper Small EN (int8)`                     | `sherpa-onnx` | `whisper`           | Offline   | English                             | No                     | MIT                                     |
| `Whisper Small Multilingual (int8)`           | `sherpa-onnx` | `whisper`           | Offline   | [42](#whisper-multilingual)         | No                     | MIT                                     |
| `Whisper Medium EN (int8)`                    | `sherpa-onnx` | `whisper`           | Offline   | English                             | No                     | MIT                                     |
| `Whisper Medium Multilingual (int8)`          | `sherpa-onnx` | `whisper`           | Offline   | [42](#whisper-multilingual)         | No                     | MIT                                     |
| `Whisper Large v3 Turbo (int8)`               | `sherpa-onnx` | `whisper`           | Offline   | [42](#whisper-multilingual)         | No                     | MIT                                     |
| `Distil-Whisper Large v3.5 EN (int8)`         | `sherpa-onnx` | `whisper`           | Offline   | English                             | No                     | MIT                                     |
| `Parakeet TDT 0.6B v3 (int8)`                 | `sherpa-onnx` | `nemo_transducer`   | Offline   | [25 European](#parakeet-tdt-06b-v3) | **Yes**, on by default | CC-BY-4.0 (attribution)                 |
| `Qwen3-ASR 0.6B (int8)`                       | `sherpa-onnx` | `qwen3_asr`         | Offline   | Not declared in the catalog         | No                     | Apache-2.0                              |
| `Streaming Zipformer EN (int8)`               | `sherpa-onnx` | `online_transducer` | Streaming | English                             | Yes, when turned on    | Apache-2.0                              |
| `Streaming Paraformer Bilingual zh-en (int8)` | `sherpa-onnx` | `online_paraformer` | Streaming | English, Chinese (`en`, `zh`)       | No                     | FunASR Model Licence v1.1 (attribution) |

Rough guidance:

- Pick a **streaming** model when the player should see words appear while speaking. Only English and Chinese/English stream.
- Pick **Parakeet TDT 0.6B v3** for game-specific names and for most European languages.
- Pick a **Whisper** model for the widest language list. Larger Whisper models are more accurate and slower. The `EN` builds take English only.

For on-disk sizes, see [Estimate Memory Footprint](https://docs.tryllengine.com/v0.10/how-to/estimate-memory-footprint.html).

Attribution is required for some weights

- Whisper and Distil-Whisper are MIT.
- Parakeet TDT 0.6B v3 is CC-BY-4.0. Credit NVIDIA.
- Qwen3-ASR 0.6B is Apache-2.0.
- Streaming Zipformer EN is Apache-2.0.
- Streaming Paraformer is under the FunASR Model Licence v1.1. Credit the source and keep the original model name.
- Silero VAD is MIT.

If you ship a recognizer in a game, include its attribution in your credits or third-party notices.

______________________________________________________________________

## Offline versus streaming

The catalog decides the mode. A variant with `"stt_recognizer": "online"` is streaming. Every other STT variant is offline.

|                                                                           | Offline                                            | Streaming                                          |
| ------------------------------------------------------------------------- | -------------------------------------------------- | -------------------------------------------------- |
| Shipped models                                                            | Whisper, Distil-Whisper, Parakeet, Qwen3-ASR       | Streaming Zipformer EN, Streaming Paraformer zh-en |
| Segmentation                                                              | Silero VAD                                         | The recognizer's built-in endpointer               |
| `Partial` updates                                                         | None                                               | Yes, revisable                                     |
| `SpeechStart` update                                                      | Yes                                                | None                                               |
| Feel                                                                      | Text arrives after each spoken segment             | Text arrives while the player speaks               |
| VAD settings (`vad_threshold`, `vad_min_silence_ms`, `vad_speech_pad_ms`) | Used                                               | Ignored                                            |
| Session locale                                                            | See [Languages](#languages-and-the-session-locale) | Checked only; the weights fix the language         |

The streaming endpointer uses fixed thresholds. It closes a segment after 0.5 s of silence that follows speech. After 2.4 s of silence with no speech it sends an empty `SegmentFinal`, a heartbeat that keeps a hands-free turn open. You cannot change these values.

Both modes work with push-to-talk and hands-free. For what closes the turn in each case, see [How the two combine](https://docs.tryllengine.com/v0.10/concepts/stt-and-voice-input.html#how-the-two-combine).

______________________________________________________________________

## Languages and the session locale

Each catalog entry may declare its languages in `languages`. `ListModels` returns them in `ModelInfo.languages`, so a client can build a language menu from installed models.

The session locale (`CreateSessionRequest.locale`) applies to voice input in two ways:

1. **Check.** At `CreateVoiceInput`, if the model declares languages and the session language is not among them, the request fails with [`3017 ModelLanguageUnsupported`](https://docs.tryllengine.com/v0.10/reference/error-codes.html#3xxx-agent). A model that declares no languages is never checked.
1. **Hint.** Whisper models also receive the session language as a decoding hint. Without a locale, a multilingual Whisper model detects the language itself, and an `EN` model assumes English.

| Model                      | Checked against the locale | Receives the language hint |
| -------------------------- | -------------------------- | -------------------------- |
| Whisper, Distil-Whisper    | Yes                        | Yes                        |
| Parakeet TDT 0.6B v3       | Yes                        | No                         |
| Qwen3-ASR 0.6B             | No (declares no languages) | No                         |
| Streaming Zipformer EN     | Yes                        | No                         |
| Streaming Paraformer zh-en | Yes                        | No                         |

A streaming model's language is fixed by its weights. With streaming, the model choice is the language choice. For the whole localization workflow, see [Ship an Agent in Another Language](https://docs.tryllengine.com/v0.10/how-to/ship-in-another-language.html).

### Whisper Multilingual

`Whisper Base Multilingual (int8)`, `Whisper Small Multilingual (int8)`, `Whisper Medium Multilingual (int8)` and `Whisper Large v3 Turbo (int8)` declare these 42 codes:

`ar`, `bg`, `bn`, `ca`, `cs`, `da`, `de`, `el`, `en`, `es`, `et`, `fa`, `fi`, `fr`, `he`, `hi`, `hr`, `hu`, `id`, `it`, `ja`, `ko`, `lt`, `lv`, `ms`, `nl`, `no`, `pl`, `pt`, `ro`, `ru`, `sk`, `sl`, `sr`, `sv`, `ta`, `th`, `tr`, `uk`, `ur`, `vi`, `zh`.

### Parakeet TDT 0.6B v3

`Parakeet TDT 0.6B v3 (int8)` declares these 25 codes:

`bg`, `cs`, `da`, `de`, `el`, `en`, `es`, `et`, `fi`, `fr`, `hr`, `hu`, `it`, `lt`, `lv`, `mt`, `nl`, `pl`, `pt`, `ro`, `ru`, `sk`, `sl`, `sv`, `uk`.

______________________________________________________________________

## Hotwords

A hotword list biases the recognizer toward game-specific words, such as NPC and spell names. Biasing needs beam-search decoding, and only the transducer families have it. For the full recipe, see [Bias Voice Input with Hotwords](https://docs.tryllengine.com/v0.10/how-to/use-voice-input-hotwords.html).

| Model                                         | Hotword biasing                                         |
| --------------------------------------------- | ------------------------------------------------------- |
| `Parakeet TDT 0.6B v3 (int8)`                 | On by default (catalog `stt_hotwords_score` is `4.0`)   |
| `Streaming Zipformer EN (int8)`               | Off by default. Turn it on with **Hotword biasing: On** |
| Every Whisper model                           | Not available                                           |
| `Qwen3-ASR 0.6B (int8)`                       | Not available                                           |
| `Streaming Paraformer Bilingual zh-en (int8)` | Not available                                           |

**Hotword biasing** is a per-model setting in the Model Manager (Default, Off, On). From code it is a per-model hint on the session. It is fixed when the model loads, so it applies to every voice input on that model in the session.

If you pass a hotword list (`hotwords_storage_path`) to a model that is not biasing, `CreateVoiceInput` fails with [`4106 HotwordBiasDisabled`](https://docs.tryllengine.com/v0.10/reference/error-codes.html#4xxx-inference-workflow). This covers a model that cannot bias and a model whose biasing is off. Drop the list or pick a model that biases.

______________________________________________________________________

## Voice activity detection

Offline recognizers use **Silero VAD** to find speech in the audio stream. The catalog entry is `Silero VAD` (`model_type: "vad"`, MIT). The server uses the VAD model named by `default_vad_model` in [Server Configuration](https://docs.tryllengine.com/v0.10/reference/server-configuration.html#speech-stt). The default is `"Silero VAD"`.

Silero VAD is a separate download. Downloading an STT model does not download it, and loading an STT model does not download it either.

When an offline model loads, the server looks for the VAD files on disk. If they are missing, the model loads anyway in **flush-only** mode, and the server logs a warning. In flush-only mode:

- No `SpeechStart` or `SegmentFinal` updates arrive.
- The whole utterance is decoded once, when it ends.
- A hands-free utterance does not close on silence. It closes on `EndUtterance` or when `max_utterance_ms` runs out.

The VAD files are looked up when the STT model loads. If you download Silero VAD after an offline model is loaded, the change takes effect the next time that model loads.

Streaming recognizers never use Silero VAD.

Ship Silero VAD with the game

Packaged players cannot download models. In the Unity and Unreal Model Manager, mark `Silero VAD` as **Production** next to any offline STT model. The build check flags a project that references an STT model with no VAD model marked Production. See [Build registration](https://docs.tryllengine.com/v0.10/reference/model-management.html#build-registration).

______________________________________________________________________

## What cannot be pinned on its own

`LoadModelRequest` pins an STT model like any other model. The pinned recognizer uses the session's locale and hotword setting, so it is the same one `CreateVoiceInput` asks for later. See [Pin and Unpin Models](https://docs.tryllengine.com/v0.10/how-to/pin-and-unpin-models.html).

A VAD entry cannot be loaded, pinned or unloaded on its own. It is download and configuration data only. Offline STT loads it as a dependency. `LoadModelRequest` for `Silero VAD` returns [`6005 ModelResolutionFailed`](https://docs.tryllengine.com/v0.10/reference/error-codes.html#6xxx-model-download). Download it with the Model Manager or `DownloadModelRequest`, then pin the offline STT model instead.

______________________________________________________________________

## Related

- [STT and Voice Input](https://docs.tryllengine.com/v0.10/concepts/stt-and-voice-input.html) — the utterance lifecycle, push-to-talk versus hands-free, and transcript updates.
- [Use Voice Input](https://docs.tryllengine.com/v0.10/how-to/use-voice-input.html) — task walkthrough for every client.
- [Bias Voice Input with Hotwords](https://docs.tryllengine.com/v0.10/how-to/use-voice-input-hotwords.html) — phrase lists, scores and the Hotword biasing control.
- [TTS Models](https://docs.tryllengine.com/v0.10/reference/tts-models.html) — the speech-output counterpart of this page.
- [Model Management](https://docs.tryllengine.com/v0.10/reference/model-management.html) — `models.json` schema, download and load lifecycle.
- [Error Codes](https://docs.tryllengine.com/v0.10/reference/error-codes.html) — `3017`, `4100`–`4109` and `6005`.
