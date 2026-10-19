# Design a Voice Clip with Qwen3-TTS

Make a reference clip for a character without recording an actor. Qwen3-TTS **VoiceDesign** creates a voice from a text description, such as "a gruff man in his fifties with a low, gravelly voice". You render a short line in that voice, save it as a WAV, and the agent's voice profile [clones](https://docs.tryllengine.com/v0.9/how-to/clone-a-voice.html) it at runtime.

VoiceDesign runs **outside Tryll**, while you author content. Tryll does not use it to speak at runtime: a description-based model samples a new voice on every call, and Tryll synthesizes a reply sentence by sentence, so the voice would change in the middle of a reply. A clip fixes the voice once, and the cloning model then keeps it the same in every sentence.

Prerequisites

- A cloning TTS model for runtime: `Qwen3-TTS 12Hz 1.7B Base (Q4_K_M)` or `Pocket TTS (int8)`. See [Clone a Voice](https://docs.tryllengine.com/v0.9/how-to/clone-a-voice.html).
- A web browser for the hosted demo, or Python if you run VoiceDesign on your own machine.

## 1. Open VoiceDesign

- **Hosted demo.** Open the [Qwen3-TTS demo](https://huggingface.co/spaces/Qwen/Qwen3-TTS) on Hugging Face and stay on the **Voice Design** tab. It has three inputs: **Text to Synthesize** (the words of the clip), **Language**, and **Voice Description** (who is speaking and how).
- **On your own machine.** Install Qwen's `qwen-tts` Python package and run `qwen-tts-demo Qwen/Qwen3-TTS-12Hz-1.7B-VoiceDesign`. It opens the same interface locally. See the [Qwen3-TTS repository](https://github.com/QwenLM/Qwen3-TTS) for setup.

The other two tabs are not needed here. **Voice Clone (Base)** is the same cloning that Tryll runs. **TTS (CustomVoice)** speaks with Qwen's preset speakers.

## 2. Write the voice description

The description decides the voice. Vague words give a generic voice, and a different one each time. These rules come from Alibaba Cloud's [Voice Design guide](https://help.aliyun.com/en/model-studio/voice-design-user-guide):

- **Be specific.** Use words for qualities you can hear, such as *deep*, *husky*, *crisp*, *nasal*, *fast-paced*. Avoid *nice*, *normal*, *good*.

- **Combine several dimensions.** "Female voice" alone says almost nothing. Cover most of these:

  | Dimension            | Examples                                                                               |
  | -------------------- | -------------------------------------------------------------------------------------- |
  | Gender               | male, female                                                                           |
  | Age                  | child, teenager, young adult, middle-aged, elderly, or an age such as "in her sixties" |
  | Pitch                | high, medium, low; "slightly low"                                                      |
  | Speed                | fast, medium, slow                                                                     |
  | Timbre               | resonant, husky, gravelly, breathy, crisp, mellow                                      |
  | Accent               | Scottish accent, light Eastern European accent                                         |
  | Emotion and delivery | calm, cheerful, irritated, fearful, whispering                                         |
  | Role                 | tavern keeper, army sergeant, news anchor, storyteller                                 |

- **Describe the sound, not your taste.** "Warm, slow, slightly breathy" works. "A voice players will love" does not.

- **Do not ask for a real person.** Describe the qualities instead of naming an actor or a celebrity.

- **Keep it short.** One or two sentences. Drop repeated synonyms and filler.

- **Write it in English or Chinese.** The text to synthesize can be in any of the model's languages.

Some examples:

| Too vague                     | Better                                                                                                                       |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| "Old man."                    | "A man in his seventies with a low, gravelly voice, speaking slowly and kindly, like a village storyteller."                 |
| "Female warrior, cool voice." | "A woman in her thirties with a firm, slightly low voice and a light Scottish accent, speaking briskly with calm authority." |
| "A merchant."                 | "A cheerful middle-aged man with a bright, slightly nasal voice, speaking fast and eagerly, like a market trader."           |

For a character who will need **several moods**, write the description in two parts: the **identity** (gender, age, pitch, timbre, accent) and the **delivery** (the emotion). Keep the identity part word for word, and change only the delivery for each mood. See [Add Emotions to a Voice](https://docs.tryllengine.com/v0.9/how-to/add-emotions-to-a-voice.html).

## 3. Write the text to synthesize

- **The model speaks the text word for word.** Put stage directions in the description, never in the text.
- **Write a line the character would say, in the mood you want.** The words help the delivery: an angry clip is easier to get from an angry line.
- **Use the language the character speaks in the game.** Set **Language** to match, or leave it on **Auto**. Pocket TTS speaks English only.
- **Match the clip length to the runtime model.** Qwen3-TTS Base needs **3–10 seconds**. Pocket TTS works best with **10–20 seconds**, so write two or three sentences. Render the whole clip in one go: takes joined from separate renders can sound like different people.

## 4. Render several takes, keep the best

The same description gives a slightly different voice on every render. Render a few takes, and keep the one that fits the character best. Listen for:

- a voice that stays the same from the first word to the last;
- clean audio with no clicks, warbling or cut-off words;
- speech that starts right away, without a long silence.

If no take fits, change the description, not just the seed. Add the dimension that is off: the age, the pitch, or the speed.

## 5. Save the clip and use it

1. Download the take with the download button on the **Generated Audio** player.
1. Convert it to **mono 16-bit PCM WAV** (24 kHz is ideal) and trim any silence at the start. The Audacity and ffmpeg steps are in [Clone a Voice → Record a reference clip](https://docs.tryllengine.com/v0.9/how-to/clone-a-voice.html#1-record-or-choose-a-reference-clip).
1. Put it in the storage folder, for example `<storage root>/voices/innkeeper/neutral.wav`.
1. Point the voice profile's **Reference Voice** at it, as in [Clone a Voice → Point the agent at the clip](https://docs.tryllengine.com/v0.9/how-to/clone-a-voice.html#3-point-the-agent-at-the-clip).

Licence and rights

Check the licence of the Qwen3-TTS model and the terms of the service you render with before you ship a generated clip in a product. Do not describe a real person's voice.

## Related

- [Clone a Voice from an Audio Sample](https://docs.tryllengine.com/v0.9/how-to/clone-a-voice.html) — how the clip is used at runtime
- [Add Emotions to a Voice](https://docs.tryllengine.com/v0.9/how-to/add-emotions-to-a-voice.html) — one clip per mood, switched with delivery modes
- [Reference: TTS Models](https://docs.tryllengine.com/v0.9/reference/tts-models.html) — Qwen3-TTS Base and Pocket TTS
- [Alibaba Cloud: Voice Design guide](https://help.aliyun.com/en/model-studio/voice-design-user-guide) — how to write a voice description
- [Qwen3-TTS on GitHub](https://github.com/QwenLM/Qwen3-TTS) — the models, local setup, and example descriptions
