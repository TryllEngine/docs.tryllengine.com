# Localize Prompts with Your Localization System

Store a prompt as a Text ID in the localization system you already ship, and have Unity or Unreal resolve it to plain text before the agent is created. The server still receives a string. There is no Tryll string table.

## 1. Tick Localized and set the table

On a workflow asset, set **Text Table** to the table that holds your prompts. On each prompt field (system prompt, template, instruction, acknowledge text, TTS delivery instruction, and a ClassifyIntent entry's prompt), tick **Localize text**. The box is then a Text ID, for example `tobin.system`. The inline text stays in the asset; only the resolved string is sent.

Leave the box unticked when that field should stay inline. A Summarize template that already says `Write in {{language}}` is a good candidate to leave inline: the session language fills the key, and the template itself does not need a translation.

Give prompt authors their own table, and keep it out of machine translation. A prompt is instructions to a model, not UI copy. Tryll does not enforce that split.

## 2. Point the project at a text provider

The provider answers `GetText(table, id)` and reports the game's current language. When one is set, that language is the session language unless you pass a locale into `CreateSession`. An empty string passed in still means no session language.

### Unity

Assign a `TryllTextProviderAsset` under **Project Settings → Tryll Client → Text Provider**. The same asset is used in the editor preview and in Play.

- **Unity Localization** (`com.unity.localization`), when that package is installed: create a **Unity Localization Text Provider**. Tryll reads the raw table entry. Turn Smart String formatting off for prompt entries. `{}` placeholders and Mustache `{{ }}` must not be nested in the same string; a Smart String would format the braces before the server sees the Mustache tags.
- **JSON Text Provider**, for a demo or a prototype: one JSON `TextAsset` per language, each a flat object `{ "id": "text" }`. Lookup tries the current locale, then the language subtag (`de-DE` then `de`), then the first other file that has the key. That last step uses another language, so it logs a warning once per key. A file that is not valid JSON logs an error with the line and column, and the inspector shows the same reason. Call `SetLocale("de")` to switch at runtime; the asset's **Locale** field is the start language and does not change in Play mode.
- **Your own class**: implement `ITryllTextProvider`. A code-only provider works in Play. The inspector preview also needs `ITryllTextProviderEditor` (`ListLocales`, `ListIds`, `GetTextForPreview`, `OpenInEditor`). Without that, the field says it has no editor support.

A provider set on `TryllClient.TextProvider` from code wins over the settings asset. Setting that property to null turns resolution off even when the settings asset has a provider.

Raise `LocaleChanged` on the main thread. Tryll sends requests from that event.

I2 Localization, in about ten lines:

```
public sealed class I2TryllTextProvider : TryllTextProviderAsset
{
    public override string CurrentLocale => I2.Loc.LocalizationManager.CurrentLanguageCode;

    public override Task<string> GetTextAsync(string table, string id)
    {
        string text = I2.Loc.LocalizationManager.GetTranslation(id);
        return Task.FromResult(string.IsNullOrEmpty(text) ? null : text);
    }

    void OnEnable() => I2.Loc.LocalizationManager.OnLocalizeEvent += RaiseLocaleChanged;
    void OnDisable() => I2.Loc.LocalizationManager.OnLocalizeEvent -= RaiseLocaleChanged;
}
```

Shared snippets (`{{text.some_id}}` inside a prompt) are the provider's job in this version. Resolve them before you return the string.

### Unreal

**Project Settings → Tryll Client → Text Provider** is **None** by default, so the Tryll **Locale** field sets the session language, as before. Pick `UTryllStringTableTextProvider` to resolve Text IDs from Unreal String Tables. It reads `FText::FromStringTable` and follows the game culture. While it is set, that culture is the session language and wins over the **Locale** field.

Pick the workflow's **Text Table** with the String Table asset picker. The asset reference makes the cooker package the table, and Tryll loads it before the first lookup, so a table that nothing else uses still works in a packaged game. A custom `UTryllTextProvider` subclass may ignore the table and use only the key. Implement `GetText` and `GetCurrentLocale`, and call `UTryllSubsystem::NotifyTextLocaleChanged` when the language changes. Override `SupportsEditorPreview` when the details panel should show a preview.

Test a runtime language switch with **Play → Standalone Game**, not in the viewport. Play in Editor shares one culture with the editor, so `SetCurrentCulture` switches the editor's own language too, and the editor loads no game translations unless `Localization.ForceLoadGameLocalizationInEditor` is set.

### C++, Python, and Java

Those clients have no provider seam. Resolve the string in your game and assign it:

```
params->SystemPrompt = Loc.Get(TEXT("bartender.system"));
```

```
p.system_prompt = loc.get("bartender.system")
```

## 3. Check the texts before you play

Under a ticked field the inspector shows a key list (typing a new key is allowed), **Open**, and a language switcher.

- **Found** — the first lines of the entry, with `{{ }}` marked.
- **Missing in this language** — red. A fallback from another language is shown and marked as fallback. Language fallback is the provider's job; Tryll does not invent one at send time.
- **Missing in some languages** — yellow. **Check texts** is the full matrix across every language the provider lists.
- **Empty ID** — red, "Enter a Text ID".
- **No editor provider** — grey. The runtime provider has no preview API.

**Check texts** on the workflow runs every Text ID against every language the provider lists. A missing cell is red.

Creation fails on the client before anything is sent when a ticked field has an empty ID, no provider is set, the provider returns no text, or the provider throws. The error names the asset, node, field, table, and ID (`InvalidParamValue`, 3007). There is no silent fallback to the inline string.

`ChangeParams` follows the same rules. Params with a ticked field are resolved first, so you can change a Text ID at runtime. With an asynchronous provider, the change can reach the server a frame later.

A Dialog Lab override on a ticked field is the text under test: it wins, and the field is unticked for that run.

Resolved text is plain text. The server's existing checks still apply: Mustache, `{{intents_block}}`, and the rest.

## 4. Change language on a live agent

When the provider raises its language-changed event, Tryll sets the session locale and re-resolves ticked fields that are not structural, then sends `ChangeAgentParam` for each live agent. A turn that is still running returns `AgentBusy` (3004); the update is sent when the turn ends, before your turn-complete handler runs. `AgentBusy` here is expected and is not reported as an error.

- **Your runtime changes stay.** The update starts from the agent's current params, so values you set with `ChangeParams` are kept. Only nodes whose text changed are sent.
- **One agent is all or nothing.** If any Text ID of an agent is missing in the new language, nothing is sent to that agent, and it keeps its previous prompts in full. Unity raises `TryllClient.LocalizedTextFailed`; Unreal broadcasts `OnLocalizedTextFailed` on the Tryll subsystem. The session language has already changed, so recreate the agent or switch back. In Unreal, a server that rejects an update raises the same event.
- **Wait for the update before work that must see it (Unreal).** The Tryll subsystem broadcasts `OnLocalizedTextApplied`. It fires once per agent, when the server has acknowledged every prompt that changed, or right away when none changed. A busy agent gets it after its turn ends. If the language changes again before the acknowledgements arrive, only the latest change fires it. Start a KV prefill, a conversation reset, or your own `ChangeParams` from this event, not from the language change itself: those run against the old prompts if they reach the server first.
- **An agent created during the switch** is updated as soon as its creation completes.
- **An explicit session language wins.** If you passed a locale to `CreateSession`, a later provider change does not touch the session language. Prompts still follow the provider.

| Reaches a live agent                                               | Needs the agent recreated                                               |
| ------------------------------------------------------------------ | ----------------------------------------------------------------------- |
| Voice language and the `{{language}}` keys, via the session locale | A ticked ClassifyIntent entry prompt (the `intents` list is structural) |
| Ticked non-structural prompt fields, from the next turn            | Files already bound (a knowledge base, canned lines)                    |
|                                                                    | A voice input that is already open — close it and open it again         |

There is no restart API. Recreating an agent to pick up structural text or files that are already bound is the same save-and-restore you already use for dialog state: export the dialog, destroy the agent, create it again, and import. See [Save and Load Agent State](https://docs.tryllengine.com/v0.9/how-to/save-and-load-agent-state.html).
