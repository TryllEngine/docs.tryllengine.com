# Ship an Agent in Another Language

Everything Tryll offers for localization, in the order you will need it: choose languages your models can serve, set one language for the session, ship translated content files, and make the model actually write in that language.

The language setting does not translate anything

Setting a language makes an NPC **speak** German. It does not make the model **write** German — that comes from your prompts, your content files, and the model you chose. Sections 4 and 5 are the load-bearing ones; skip them and you get a German voice reading English words.

## 1. Choose languages your models can serve

Open *Window ▸ Tryll ▸ Model Manager* (Unity) or the Model Manager panel (Unreal), connect, and register the models you ship. Each model's detail pane lists the languages it declares and whether it covers your project language.

Registering caches those languages into the project's model manifest — which is why the language picker and the inspector warnings keep working with no server running.

Three facts the catalog will tell you, which decide your shippable locale set:

- **Supertonic 3** speaks 31 languages from one loaded model. Switching costs nothing at runtime.
- **Pocket TTS** — the voice-*cloning* family — is **English-only**. Cloned character voices cannot go multilingual.
- The shipped **streaming** STT choices cover English, bilingual zh-en, and Russian. Other languages use an offline recognizer.

**The voice stack caps your locale set, not the language model.** A 1–4B model reads almost anything; your microphone and speaker decide what you can ship.

From code

`ListModels` reports a `languages` list per model. A language is offerable only if *every* model in your pipeline covers it, so the answer is an intersection — `TryllLanguages` (Unity) and `UTryllLanguages` (Unreal, Blueprint-callable) do that for you:

```
var (models, err) = await TryllClient.Instance.RequestListModelsAsync();
var codes = TryllLanguages.Offerable(models,
    "Supertonic 3 (int8)", "Whisper Base Multilingual (int8)");

foreach (var code in codes)
    dropdown.options.Add(new Dropdown.OptionData(
        TryllLanguages.DisplayName(code)));   // "de" → "German"
```

A model that declares nothing is skipped rather than treated as restrictive — undeclared means "the catalog didn't say", not "supports nothing". Use `HasLanguageInfo` to tell "no overlap" apart from "no data" before showing a player an empty menu.

## 2. Set the language

Pick it in *Project Settings ▸ Tryll Client ▸ Language*. The list comes from your registered models, annotated with how many catalog models can serve each one — `German (de) — 7 models` — so you can spot a language that hangs off a single model before committing to it. New projects start on English.

Every agent created afterwards uses it automatically, including agents created later in the session.

From code, you can override per session:

```
// Omit locale (pass null) and the project setting is used.
// Storage is not a session argument — the launcher sets --storage-root.
TryllClient.Instance.CreateSession(
    engines, settings.GameName, locale: "de-DE");
```

```
Subsystem->CreateSession(TEXT("my-game"), TEXT("de-DE"));

// Omit Locale to use the project setting. To explicitly opt out:
Subsystem->CreateSession(
    TEXT("my-game"), TEXT(""), /*bUseProjectLocaleWhenEmpty=*/false);
```

```
TryllClient::SessionConfig cfg;
cfg.locale = "de-DE";
client.CreateSession(cfg);
```

```
client.create_session(locale='de-DE')
```

Tags are BCP-47: `de`, `de-DE`, `pt-BR`, `zh-Hans-CN`. A malformed tag is rejected with `InvalidLocale` (2004) rather than silently ignored.

### Changing it mid-session

```
TryllClient.Instance.RequestSetSessionLocale("pt-BR");   // "" clears it
```

```
Subsystem->SetSessionLocale(TEXT("pt-BR"));              // "" clears it
```

```
client.SetSessionLocale("pt-BR");                        // "" clears it
```

```
client.set_session_locale("pt-BR")                       # "" clears it
```

| Follows the change, from each agent's next turn | Does not                                        |
| ----------------------------------------------- | ----------------------------------------------- |
| Speech-output language on every agent           | A knowledge base already bound to a node        |
| The `{{language}}` template keys                | Canned lines / classifier prompts already bound |
| File resolution for the *next* load             | A voice input already open                      |

The right-hand column is structural — those parameters are fixed at agent creation. So a live switch is the right tool for the **voice and prompt** half; changing *authored content* still means recreating agents. Most games treat a language change as "set it, then reload the scene", which gets both halves.

It is also not turn-atomic: switch mid-sentence and that one line may be written in the old language and spoken in the new one. Send it between turns.

A live switch does not re-check existing agents. If their TTS model does not declare the new language, no `3017` is raised — the node just speaks with the model's own handling of that tag. Check coverage before switching live (for example with `TryllLanguages.Offerable`).

## 3. Ship translated content files

Translations live in a **`loc/<language>/` folder beside the file you link**.

```
StorageData/
  deathonset/
    lore.json                 ← you link this; it is the default content
    canned.txt
    loc/
      de/lore.json            ← used when the session runs in German
      pt-BR/lore.json
  shared/
    intent_ids.json           ← no loc/ folder = language-neutral, on purpose
```

Resolution, most specific first:

```
deathonset/loc/pt-BR/lore.json      ← used if it exists
deathonset/loc/pt/lore.json         ← else if it exists
deathonset/lore.json                ← else: the file you linked
```

Three consequences worth internalising:

- **You always link the default file.** Never a path inside `loc/`. The graph names one path and the shipped folder decides which language answers. If you browse to a translation by mistake, the editor offers to link the default instead.
- **A folder with no `loc/` is language-neutral.** That is the whole declaration — no flag, no metadata. Intent IDs, guardrail patterns and other config just sit there and are shared by every language.
- **Fallback is per file.** Ship `loc/de/lore.json` before `loc/de/canned.txt` exists and only the knowledge base switches. A partially translated game works.

This covers every file-backed parameter: knowledge bases and their `.usearch`/`.bm25` sidecars, canned responses, guardrail patterns, intent maps, voice-input hotwords, and reference-voice WAVs.

`loc` is a reserved folder name

Do not use `loc/` for your own content inside a storage folder. A project that wants symmetry may also ship `loc/en/`, but nothing requires it — the linked file *is* the default.

You may not need to translate knowledge bases at all

With a multilingual embedding model, `Retrieve` and `ClassifyIntent` match a German question against an English knowledge base, so the model only has to *read* English and *write* German. That removes most of the translation cost. It only helps the dense leg — BM25 cannot match across languages, so `Hybrid` retrieval degrades to dense-only.

## 4. Make the model write the language

The generated half has no string to translate, so it needs a prompt strategy. Three options, in increasing order of quality:

**a. Ask, from the template.** Three keys are available wherever Mustache rendering is supported — on `Generate`, `GenerateAndSpeak`, `ToolCall` and `Transform`:

| Key                 | Renders                                                        |
| ------------------- | -------------------------------------------------------------- |
| `{{language}}`      | `German` — the English language name, which models follow best |
| `{{locale}}`        | `de-DE` — the full tag                                         |
| `{{language_code}}` | `de` — the bare code                                           |

```
{{#language}}Always reply in {{language}}.{{/language}}
{{user_message}}
```

The section collapses to nothing when no language is set, so the same template works unlocalized.

**b. Author the prompt in the target language.** Writing the system prompt *in* German reduces mid-response drift far more reliably than an English prompt asking for German. Use a locale-specific workflow asset or set the mutable prompt/template params from your game's localization system. The `loc/` lookup in §3 applies only to file-backed storage params; it does not automatically localize strings authored on the workflow asset.

**c. Use a different model per language.** A node's `model_name` can differ per locale. A model genuinely strong in the target language beats prompt engineering on a weak one.

**The criticality gradient.** Critical narrative lines → `CannedResponse` + `Speak` with fully translated text (zero model risk). Constrained barks → a per-language `grammar`. Ambient chatter → free generation. This lets you ship a locale where the model is mediocre without shipping mediocre *important* lines.

## 5. Voice in and out

**TTS.** Every `GenerateAndSpeak` and `Speak` node uses the session language as its `tts_lang` default. A node that authored its own **Tts Lang** still wins — use that only to override one character (a French witness in a German playthrough), never to set the game's language.

**STT.** A multilingual recognizer (Whisper, Canary) receives the language as a hint. A recognizer whose weights fix its language ignores it: with STT the model choice *is* the language choice.

Speaking a language the recognizer was not told about

A wrong language hint does not fail loudly — Whisper returns confident nonsense. This is why Tryll only passes the hint to models whose catalog declares that language, and refuses the combination outright rather than guessing.

## 6. Errors you will see

| Error                             | Means                                                                       | Fix                                                                                  |
| --------------------------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| `InvalidLocale` (2004)            | The tag is not `language[-script][-region]`                                 | Use `de`, `de-DE`, `pt-BR`                                                           |
| `ModelLanguageUnsupported` (3017) | A node would rely on the session language but its model does not declare it | Pick a covering model, set the node's own `tts_lang`, or change the session language |

`3017` is raised at **agent creation** for TTS nodes and at **CreateVoiceInput** for STT — the same mismatch the inspector warned about, at the point it starts to matter. Two deliberate exemptions: a node with its own `tts_lang` is never checked (you opted out of the session language), and a model that declares *no* languages is never checked (undeclared means unknown, not unsupported).

## 7. Never translate these

Translating identifiers breaks routing **silently** rather than loudly:

`intents_ids` · slot names (`output_name`, `input`) · node names · exit names (`*_exit`) · variable names and `__MARKER__` names · storage names · model catalog names · tool names and parameter names.

Related: branch on variables or intent IDs, never on literal model output — `BranchParams.values` matched against text is a localization trap.

## What to expect

- **Token cost rises.** Non-English text runs roughly 2–4× the tokens for the same content, which changes `context_size` sizing, when history starts being trimmed, and per-turn latency. A graph tuned in English can degrade in Japanese with no visible cause.
- **Writing quality drops before reading quality does.** Small on-device models understand a language long before they write it well. If prose is poor, the fix is §4b or §4c — not more instructions.
- **Most studios ship the AI feature in fewer locales than the game.** That is a legitimate answer, and usually the right one.

## See also

- [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.8/how-to/add-voice-to-an-agent.html)
- [Clone a Voice from an Audio Sample](https://docs.tryllengine.com/v0.8/how-to/clone-a-voice.html)
- [Use Voice Input](https://docs.tryllengine.com/v0.8/how-to/use-voice-input.html)
- [Ship Storage Folders for Builds](https://docs.tryllengine.com/v0.8/how-to/ship-storage-folders-for-builds.html)
- [Use Mustache Templates](https://docs.tryllengine.com/v0.8/how-to/use-mustache-templates.html)
- [TTS Models](https://docs.tryllengine.com/v0.8/reference/tts-models.html)
- [Error Codes](https://docs.tryllengine.com/v0.8/reference/error-codes.html)
