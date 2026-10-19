# TTS Models

Tryll synthesizes speech through two engines:

- [`SherpaOnnx`](https://docs.tryllengine.com/v0.9/concepts/models-and-inference-engines.html) hosts **Supertonic 3** and **Pocket TTS**.
- [`LlamaCpp`](https://docs.tryllengine.com/v0.9/concepts/models-and-inference-engines.html) hosts **Orpheus** and **Maya1**. Their audio codes are decoded by a shared SNAC 24 kHz decoder that runs on ONNX Runtime on the CPU.

A third engine, `onnx-runtime`, runs the Chatterbox family. Its catalog entries are internal and are not shipped.

llama.cpp also runs **Qwen3-TTS** (`Qwen3-TTS 12Hz 1.7B Base (Q4_K_M)`, family `qwen3tts`). It is a public catalog entry. It clones its voice from a reference clip, so the voice profile's `reference_voice` is required, and it speaks 10 languages through `lang`: zh, en, de, it, pt, es, ja, ko, fr, ru.

This page is the authoritative list of **which voices the Tryll server ships, which families it can load, and why the rest are out of reach**.

Short version: Tryll ships an **eSpeak-free** Sherpa-ONNX build so the server can be redistributed inside a proprietary game. That choice makes two Sherpa families available — **Supertonic 3** and **Pocket TTS** — and puts the rest of the Sherpa zoo out of reach. Separately, two llama.cpp SNAC voices — `Orpheus 3B FT (Q4_K_M)` and `Maya1 3B (i1-Q4_K_M)` — are public catalog entries. On Maya1, identity is the voice profile's `voice_description` and delivery is a delivery mode's `instruction`; `[emotion]` is not supported syntax.

Heavier SNAC quants (`Orpheus 3B FT (Q8)`, `Maya1 3B (Q8)`, `Maya1 3B (Q4_K_M)`) stay `audience: "internal"`. Release builds keep only `public` entries, so they are not in the shipped catalog at all.

For the catalog file format these models are declared in, see [Model Management](https://docs.tryllengine.com/v0.9/reference/model-management.html). For the agent's voice settings, see [Voice Profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html).

______________________________________________________________________

## Shipped voices

The four entries below are in the default `models.json`, are downloaded on demand, and are selected with `tts_model_name` on the agent's [voice profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html). Every speaking node of the agent ([`GenerateAndSpeak`](https://docs.tryllengine.com/v0.9/reference/nodes/generate-and-speak.html), [`Speak`](https://docs.tryllengine.com/v0.9/reference/nodes/speak.html)) uses that model. The engine is taken from the catalog — you do not pick Sherpa vs. llama.cpp in Project Settings. See [Choose where models run](https://docs.tryllengine.com/v0.9/how-to/choose-where-models-run.html).

| Catalog name             | Engine        | Family       | Voices                                                                            | Languages      | Voice cloning                                                | Weights license                             |
| ------------------------ | ------------- | ------------ | --------------------------------------------------------------------------------- | -------------- | ------------------------------------------------------------ | ------------------------------------------- |
| `Supertonic 3 (int8)`    | `sherpa-onnx` | `supertonic` | 10 built-in styles (`speaker_id` `0`–`9`)                                         | 31, via `lang` | No                                                           | OpenRAIL-M (attribution + use restrictions) |
| `Pocket TTS (int8)`      | `sherpa-onnx` | `pocket`     | Single-speaker (`speaker_id` must stay `0`)                                       | English        | **Yes** — zero-shot from a reference WAV (`reference_voice`) | CC-BY-4.0 (attribution)                     |
| `Orpheus 3B FT (Q4_K_M)` | `llama-cpp`   | `orpheus`    | 8 named presets (`speaker_id` `0`–`7`: Tara, Leah, Jess, Leo, Dan, Mia, Zac, Zoe) | English        | No                                                           | Llama 3 Community License                   |
| `Maya1 3B (i1-Q4_K_M)`   | `llama-cpp`   | `maya1`      | 2 catalog presets (`speaker_id` `0`–`1`) or a natural-language description        | English        | No — describe the voice instead                              | Apache-2.0                                  |

Both SNAC voices pull a shared **SNAC 24 kHz decoder** (~50 MB) the first time either parent is downloaded. Deleting one parent does not delete the decoder.

Pick **Supertonic 3** when you want multiple distinct voices, non-English speech, or the lowest latency. Pick **Pocket TTS** when a character needs a specific voice you can supply as an audio sample — see [Clone a Voice](https://docs.tryllengine.com/v0.9/how-to/clone-a-voice.html). Pick **Orpheus** for named English speakers plus inline tags such as `<laugh>`. Pick **Maya1** when you want to brief a voice in natural language (`voice_description`) and steer delivery in words (a delivery mode's `instruction`).

Attribution is required

None of these weights are public domain.

- Supertonic 3 is OpenRAIL-M (attribution plus use-based restrictions — no impersonation or harmful use).
- Pocket TTS is CC-BY-4.0 (attribution).
- Orpheus weights are a Llama 3 derivative and inherit the [Llama 3 Community License](https://huggingface.co/canopylabs/orpheus-3b-0.1-ft); the Orpheus inference code is Apache-2.0.
- Maya1 weights are Apache-2.0.
- The shared SNAC decoder is MIT (`hubertsiuzdak/snac`).

If you ship a voice in a game, include the model attribution in your credits or third-party notices. The engine code itself (Sherpa-ONNX, ONNX Runtime, llama.cpp) is permissively licensed and imposes no such obligation.

______________________________________________________________________

## Voice controls by family

These are the [voice profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html) fields, by family. Do not encode identity or delivery as a spoken `[emotion]` prefix — that syntax is not supported. Model-native `<tag>` tokens that a catalog lists may reach the synthesizer; unknown tags are stripped from the audio copy only and do not rewrite history.

| Profile field               | Supertonic 3                                        | Pocket TTS                                                | Orpheus                                                                                 | Maya1                                                                                  | Qwen3-TTS Base                                                                                 |
| --------------------------- | --------------------------------------------------- | --------------------------------------------------------- | --------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| `speaker_id`                | `0`–`9` (style index)                               | Must stay `0`                                             | `0`–`7` (named preset)                                                                  | `0`–`1` (catalog preset)                                                               | Must stay `0`                                                                                  |
| `reference_voice`           | Rejected                                            | Reference WAV (optional; falls back to the bundled voice) | Rejected                                                                                | Rejected                                                                               | Reference WAV (**required**; no default voice)                                                 |
| `voice_description`         | Rejected                                            | Rejected                                                  | Rejected                                                                                | Stable identity (overrides `speaker_id` when set)                                      | Rejected                                                                                       |
| Mode `instruction`          | Rejected                                            | Rejected                                                  | Rejected                                                                                | Sustained delivery while the mode is active                                            | Rejected                                                                                       |
| `lang`                      | 31 languages                                        | Rejected (English only)                                   | Rejected (English only)                                                                 | Rejected (English only)                                                                | 10 languages: zh, en, de, it, pt, es, ja, ko, fr, ru (only English is verified by listen-back) |
| `speed`, mode `speed_scale` | `0.1`–`3.0`                                         | `0.1`–`3.0`                                               | `1.0` only                                                                              | `1.0` only                                                                             | `1.0` only                                                                                     |
| Inline `<tag>` tokens       | None (every `<…>` span is stripped, with a warning) | None (every `<…>` span is stripped, with a warning)       | `<laugh>`, `<chuckle>`, `<sigh>`, `<cough>`, `<sniffle>`, `<groan>`, `<yawn>`, `<gasp>` | `<laugh>`, `<giggle>`, `<sigh>`, `<gasp>`, `<angry>`, `<whisper>`, `<cry>`, `<scream>` | None (every `<…>` span is stripped, with a warning)                                            |

"Rejected" means the voice fails with `3023 InvalidVoiceProfile` when it is set (at agent creation or `SetAgentVoice`), rather than being silently ignored. This includes a non-empty `lang` on an English-only voice. When `lang` is empty and the session locale is a language the voice does not declare (Pocket, Orpheus and Maya1 declare only `en`), the voice fails with `3017 ModelLanguageUnsupported`.

`voice_description` and `instruction` may be at most 1024 bytes and must not contain `"`, `<`, `>` or control characters (tab, CR and LF are allowed).

Orpheus, Maya1 and Qwen3-TTS synthesize **one call at a time per loaded model**: several NPCs sharing one of them wait for each other. The Sherpa voices have no such limit.

Maya1 catalog presets are starting points, not the only voices:

- `0` — *Male, 30s, American*
- `1` — *Female, 30s, American*

Set `voice_description` to something like *"Realistic female voice in the 30s with an American accent. Normal pitch, warm timbre, conversational pacing."* and the description replaces the preset. Add delivery modes with an `instruction` for delivery; `neutral` (no instruction) is the plain voice. These words map to trained delivery styles, matched as whole words: `whisper`, `angry` / `anger`, `happy` / `happily`, `sad` / `sadly`, `calm`, `excited`, `serious`, `sarcastic`, `disappointed`, `frightened` / `fearful`. Other text is passed to the model as written.

______________________________________________________________________

## Delivery modes by family

A [delivery mode](https://docs.tryllengine.com/v0.9/reference/voice-profile.html#delivery-modes) is a named way of speaking that game code selects with `SetAgentDeliveryMode`. Only three things in the current models carry a sustained mood: a reference clip (Pocket TTS, Chatterbox, Qwen3-TTS), a natural-language clause (Maya1), and the Chatterbox exaggeration scalar. Supertonic 3 and Orpheus have no emotion control, so on those a mode can only change speed, or must stay empty.

| Family                                | Base identity                                           | A delivery mode can set                                                                  | `intensity` (0–1) does                                                                                                   | Cost of switching mode                                                                  | Measured strength                                                                                                       |
| ------------------------------------- | ------------------------------------------------------- | ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Supertonic 3                          | `speaker_id` (style `0`–`9`)                            | `speed_scale` only                                                                       | scales `speed_scale`                                                                                                     | none                                                                                    | No emotion. Inline expression tags measured inert.                                                                      |
| Pocket TTS                            | `reference_voice` (default `bria.wav`)                  | `reference_voice`, `speed_scale`                                                         | scales `speed_scale` only; a clip is all or nothing                                                                      | a clip's first use computes an embedding, unless it was prepared when the voice was set | The clip's mood carries over: an angry clip raised pitch, a calm one lowered it.                                        |
| Chatterbox EN / ML / Turbo (internal) | `reference_voice` (variant default)                     | `reference_voice`, `expressiveness` (variants with an exaggeration input), `speed_scale` | lerps `expressiveness` from the profile value to the mode value                                                          | a clip's first use encodes it, unless it was prepared                                   | Exaggeration rated a weak control; the family is a fallback.                                                            |
| Qwen3-TTS 1.7B Base                   | `reference_voice` (required)                            | `reference_voice` only                                                                   | nothing                                                                                                                  | a clip's first use runs the speaker encoder, unless it was prepared                     | The clip's mood carries over. There is no instruction channel: instruction text would be read aloud, so it is rejected. |
| Orpheus 3B FT                         | `speaker_id` (preset `0`–`7`)                           | nothing: modes must be empty                                                             | nothing                                                                                                                  | none                                                                                    | Changing the preset changes who speaks, so it is not a delivery. Most inline tags measured no stronger than a fake tag. |
| Maya1 3B                              | `voice_description`, else `speaker_id` (preset `0`–`1`) | `instruction`                                                                            | picks the clause's intensity word: below `0.34` low, below `0.67` medium, else high (only when `intensity` is below `1`) | none extra; the prompt is re-read on every sentence                                     | `excited` and `angry` clearly raised pitch; `happy` and `sad` on flat text were within noise.                           |

Strength figures come from Tryll's internal listen-back measurements. Treat them as a guide to which modes are worth authoring, not as guarantees.

______________________________________________________________________

## Family support matrix

Sherpa-ONNX v1.13.2 exposes seven TTS families. Tryll accepts two of them. llama.cpp SNAC families are a separate path.

| Family                 | Engine      | Upstream examples           | Status in Tryll  | Why                                                                                                                   |
| ---------------------- | ----------- | --------------------------- | ---------------- | --------------------------------------------------------------------------------------------------------------------- |
| **Supertonic**         | Sherpa-ONNX | Supertonic 3                | ✅ **Supported** | Own tokenizer (`unicode_indexer.bin`); no eSpeak                                                                      |
| **Pocket**             | Sherpa-ONNX | Pocket TTS (Kyutai)         | ✅ **Supported** | SentencePiece vocabulary; no eSpeak                                                                                   |
| **Orpheus**            | llama.cpp   | Orpheus 3B FT               | ✅ **Supported** | SNAC tokens via llama.cpp; shared ONNX decoder                                                                        |
| **Maya1**              | llama.cpp   | Maya1 3B                    | ✅ **Supported** | Same SNAC engine as Orpheus; description + delivery controls                                                          |
| **Qwen3-TTS**          | llama.cpp   | Qwen3-TTS 12Hz 1.7B Base    | ✅ **Supported** | Talker GGUF plus an mmproj codec, run through libmtmd; reference-voice cloning                                        |
| **Kokoro**             | Sherpa-ONNX | Kokoro v0.19, v1.0          | ❌ Unavailable   | Requires eSpeak NG — see below                                                                                        |
| **Kitten**             | Sherpa-ONNX | KittenTTS nano/micro/mini   | ❌ Unavailable   | Requires eSpeak NG — see below                                                                                        |
| **VITS (incl. Piper)** | Sherpa-ONNX | Piper voices, MMS, LJSpeech | ❌ Unavailable   | Piper-style bundles require eSpeak NG. Lexicon-based VITS bundles do not, but the family is not implemented in Tryll. |
| **Matcha**             | Sherpa-ONNX | Matcha-Icefall (en, zh)     | ❌ Unavailable   | English bundles require eSpeak NG. The Chinese lexicon path does not, but the family is not implemented in Tryll.     |
| **ZipVoice**           | Sherpa-ONNX | ZipVoice, ZipVoice-Distill  | ❌ Unavailable   | No eSpeak requirement, but the family is not implemented in Tryll.                                                    |

Note the two distinct reasons for the Sherpa gaps. **Kokoro and Kitten are blocked by licensing** — they cannot be enabled without changing what Tryll redistributes. **VITS, Matcha, and ZipVoice are simply not implemented**; nothing in principle prevents their eSpeak-free bundles from being added later.

______________________________________________________________________

## Why eSpeak-dependent families are unavailable

Sherpa-ONNX performs grapheme-to-phoneme conversion for several families with [eSpeak NG](https://github.com/espeak-ng/espeak-ng), which is licensed **GPLv3-or-later**. Linking it into `tryll_server.exe` would make the server a combined work under GPLv3 — and because the server payload is bundled inside your game build, that obligation would follow you, your studio, and your publisher.

Tryll therefore builds Sherpa-ONNX with eSpeak NG **compiled out entirely**. The shipped `sherpa-onnx-c-api.dll` contains no eSpeak code and the payload contains no `espeak-ng-data`. The upside is that nothing in the Tryll server imposes a copyleft obligation on your game. The cost is that any voice whose front-end calls eSpeak cannot run, regardless of how the model's own weights are licensed.

This constraint applies to Sherpa-ONNX families only. llama.cpp SNAC voices do not use eSpeak.

A permissive model license is not enough

KittenTTS is the classic trap: its weights are Apache-2.0, which looks unrestricted, but the model is *trained on eSpeak phoneme IDs* and Sherpa-ONNX's Kitten loader requires a populated `espeak-ng-data` directory. The permissive headline license does not make the runtime dependency go away. The same reasoning excludes Kokoro, whose loader also hard-requires that directory.

______________________________________________________________________

## What happens if you try an unsupported family

A Sherpa-ONNX `models.json` variant whose `tts_family` is anything other than `supertonic` or `pocket` fails when the model is loaded — that is, at `CreateAgent` time for a graph referencing it, not silently at startup. llama.cpp SNAC families (`orpheus`, `maya1`) load when their catalog entries and the shared decoder asset are present.

| `tts_family` value                                                  | Result                                                                                         |
| ------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| `supertonic`, `pocket`                                              | Loads normally on Sherpa-ONNX                                                                  |
| `orpheus`, `maya1`                                                  | Loads normally on llama.cpp + the shared SNAC decoder                                          |
| `vits`, `piper`, `kokoro`, or empty                                 | Rejected with an explicit eSpeak/GPLv3 message                                                 |
| Any other Sherpa-ONNX value (`kitten`, `matcha`, `zipvoice`, typos) | Rejected as an unknown Sherpa-ONNX family                                                      |
| An unknown llama.cpp value                                          | Rejected: no compiled-in llama.cpp TTS engine serves it (the message lists the known families) |

Those Sherpa-ONNX cases surface as agent-creation failures; see [Error Codes](https://docs.tryllengine.com/v0.9/reference/error-codes.html). An empty Sherpa-ONNX `tts_family` is rejected rather than defaulting, because the historical default was VITS.

______________________________________________________________________

## Adding your own voice

You can add TTS entries to `models.json` — either a HuggingFace repo or a local path — using the schema in [Model Management](https://docs.tryllengine.com/v0.9/reference/model-management.html#variants-entry-fields). For Sherpa-ONNX the constraint is the family, not the source: the bundle must be a Supertonic or Pocket bundle, with the `tts_files` key set that family expects. llama.cpp SNAC voices use `tts_family` `orpheus` or `maya1` plus a shared decoder asset (`dependencies.snac_decoder`).

Sherpa bundles for both supported families are published by the Sherpa-ONNX maintainer on HuggingFace — the shipped ones under [`csukuangfj2`](https://huggingface.co/csukuangfj2), the rest of the zoo mostly under [`csukuangfj`](https://huggingface.co/csukuangfj). A Sherpa bundle from any other family will be rejected at load time even though the catalog entry itself parses.

______________________________________________________________________

## Related

- [TTS and Voice Output](https://docs.tryllengine.com/v0.9/concepts/tts-and-voice-output.html) — the streaming pipeline and audio format.
- [Voice Profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html) — every voice field, delivery modes and intensity.
- [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.9/how-to/add-voice-to-an-agent.html) — task walkthrough.
- [Clone a Voice from an Audio Sample](https://docs.tryllengine.com/v0.9/how-to/clone-a-voice.html) — Pocket TTS reference-voice workflow.
- [Model Management](https://docs.tryllengine.com/v0.9/reference/model-management.html) — `models.json` schema, download and load lifecycle.
- [`GenerateAndSpeak`](https://docs.tryllengine.com/v0.9/reference/nodes/generate-and-speak.html) and [`Speak`](https://docs.tryllengine.com/v0.9/reference/nodes/speak.html) — the nodes that produce audio.
