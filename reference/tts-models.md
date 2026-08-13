# TTS Models

Tryll synthesizes speech through the [`SherpaOnnx` engine](https://docs.tryllengine.com/concepts/models-and-inference-engines.html). This page is the authoritative list of **which Sherpa-ONNX TTS families the Tryll server can load, which it cannot, and why** — so you can tell at a glance whether a voice you found upstream will work here.

Short version: Tryll ships an **eSpeak-free** Sherpa-ONNX build so the server can be redistributed inside a proprietary game. That choice makes two families available — **Supertonic 3** and **Pocket TTS** — and puts the rest out of reach.

For the catalog file format these models are declared in, see [Model Management](https://docs.tryllengine.com/reference/model-management.html). For the node parameters that select a voice at runtime, see [TTS and Voice Output](https://docs.tryllengine.com/concepts/tts-and-voice-output.html).

______________________________________________________________________

## Shipped voices

Both entries below are in the default `models.json`, are downloaded on demand, and are selected with the `tts_model_name` parameter on [`GenerateAndSpeak`](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html) or [`Speak`](https://docs.tryllengine.com/reference/nodes/speak.html).

| Catalog name          | Family       | Voices                                      | Languages          | Voice cloning                                          | Weights license                             |
| --------------------- | ------------ | ------------------------------------------- | ------------------ | ------------------------------------------------------ | ------------------------------------------- |
| `Supertonic 3 (int8)` | `supertonic` | 10 built-in styles (`speaker_id` `0`–`9`)   | 31, via `tts_lang` | No                                                     | OpenRAIL-M (attribution + use restrictions) |
| `Pocket TTS (int8)`   | `pocket`     | Single-speaker (`speaker_id` must stay `0`) | English            | **Yes** — zero-shot from a reference WAV (`tts_voice`) | CC-BY-4.0 (attribution)                     |

Pick **Supertonic 3** when you want multiple distinct voices, non-English speech, or the lowest latency. Pick **Pocket TTS** when a character needs a specific voice you can supply as an audio sample — see [Clone a Voice](https://docs.tryllengine.com/how-to/clone-a-voice.html).

Attribution is required for both

Neither model's weights are public domain. Supertonic 3 is OpenRAIL-M (attribution plus use-based restrictions — no impersonation or harmful use); Pocket TTS is CC-BY-4.0 (attribution). If you ship either voice in a game, include the model attribution in your credits or third-party notices. The engine code itself (Sherpa-ONNX, ONNX Runtime) is permissively licensed and imposes no such obligation.

______________________________________________________________________

## Family support matrix

Sherpa-ONNX v1.13.2 exposes seven TTS families. Tryll accepts two of them.

| Family                 | Upstream examples           | Status in Tryll  | Why                                                                                                                   |
| ---------------------- | --------------------------- | ---------------- | --------------------------------------------------------------------------------------------------------------------- |
| **Supertonic**         | Supertonic 3                | ✅ **Supported** | Own tokenizer (`unicode_indexer.bin`); no eSpeak                                                                      |
| **Pocket**             | Pocket TTS (Kyutai)         | ✅ **Supported** | SentencePiece vocabulary; no eSpeak                                                                                   |
| **Kokoro**             | Kokoro v0.19, v1.0          | ❌ Unavailable   | Requires eSpeak NG — see below                                                                                        |
| **Kitten**             | KittenTTS nano/micro/mini   | ❌ Unavailable   | Requires eSpeak NG — see below                                                                                        |
| **VITS (incl. Piper)** | Piper voices, MMS, LJSpeech | ❌ Unavailable   | Piper-style bundles require eSpeak NG. Lexicon-based VITS bundles do not, but the family is not implemented in Tryll. |
| **Matcha**             | Matcha-Icefall (en, zh)     | ❌ Unavailable   | English bundles require eSpeak NG. The Chinese lexicon path does not, but the family is not implemented in Tryll.     |
| **ZipVoice**           | ZipVoice, ZipVoice-Distill  | ❌ Unavailable   | No eSpeak requirement, but the family is not implemented in Tryll.                                                    |

Note the two distinct reasons. **Kokoro and Kitten are blocked by licensing** — they cannot be enabled without changing what Tryll redistributes. **VITS, Matcha, and ZipVoice are simply not implemented**; nothing in principle prevents their eSpeak-free bundles from being added later.

______________________________________________________________________

## Why eSpeak-dependent families are unavailable

Sherpa-ONNX performs grapheme-to-phoneme conversion for several families with [eSpeak NG](https://github.com/espeak-ng/espeak-ng), which is licensed **GPLv3-or-later**. Linking it into `tryll_server.exe` would make the server a combined work under GPLv3 — and because the server payload is bundled inside your game build, that obligation would follow you, your studio, and your publisher.

Tryll therefore builds Sherpa-ONNX with eSpeak NG **compiled out entirely**. The shipped `sherpa-onnx-c-api.dll` contains no eSpeak code and the payload contains no `espeak-ng-data`. The upside is that nothing in the Tryll server imposes a copyleft obligation on your game. The cost is that any voice whose front-end calls eSpeak cannot run, regardless of how the model's own weights are licensed.

A permissive model license is not enough

KittenTTS is the classic trap: its weights are Apache-2.0, which looks unrestricted, but the model is *trained on eSpeak phoneme IDs* and Sherpa-ONNX's Kitten loader requires a populated `espeak-ng-data` directory. The permissive headline license does not make the runtime dependency go away. The same reasoning excludes Kokoro, whose loader also hard-requires that directory.

______________________________________________________________________

## What happens if you try an unsupported family

A `models.json` variant whose `tts_family` is anything other than `supertonic` or `pocket` fails when the model is loaded — that is, at `CreateAgent` time for a graph referencing it, not silently at startup.

| `tts_family` value                                      | Result                                         |
| ------------------------------------------------------- | ---------------------------------------------- |
| `supertonic`, `pocket`                                  | Loads normally                                 |
| `vits`, `piper`, `kokoro`, or empty                     | Rejected with an explicit eSpeak/GPLv3 message |
| Any other value (`kitten`, `matcha`, `zipvoice`, typos) | Rejected as an unknown family                  |

Both cases surface as agent-creation failures; see [Error Codes](https://docs.tryllengine.com/reference/error-codes.html). An empty `tts_family` is rejected rather than defaulting, because the historical default was VITS.

______________________________________________________________________

## Adding your own voice

You can add TTS entries to `models.json` — either a HuggingFace repo or a local path — using the schema in [Model Management](https://docs.tryllengine.com/reference/model-management.html#variants-entry-fields). The constraint is the family, not the source: the bundle must be a Supertonic or Pocket bundle, with the `tts_files` key set that family expects.

Bundles for both families are published under [`csukuangfj` on HuggingFace](https://huggingface.co/csukuangfj) alongside the rest of the Sherpa-ONNX model zoo. A bundle from any other family will be rejected at load time even though the catalog entry itself parses.

______________________________________________________________________

## Related

- [TTS and Voice Output](https://docs.tryllengine.com/concepts/tts-and-voice-output.html) — the streaming pipeline, audio format, and every voice parameter.
- [Add Voice Output to an Agent](https://docs.tryllengine.com/how-to/add-voice-to-an-agent.html) — task walkthrough.
- [Clone a Voice from an Audio Sample](https://docs.tryllengine.com/how-to/clone-a-voice.html) — Pocket TTS reference-voice workflow.
- [Model Management](https://docs.tryllengine.com/reference/model-management.html) — `models.json` schema, download and load lifecycle.
- [`GenerateAndSpeak`](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html) and [`Speak`](https://docs.tryllengine.com/reference/nodes/speak.html) — the nodes that produce audio.
