# Release Notes

One page per version, newest first. Each page lists the release's highlights by theme, its **breaking changes and behavior changes**, and any **new error codes** — read the breaking-changes section of every version between the one you shipped and the one you are upgrading to.

Tryll is in pre-release: expect breaking changes until the 1.0 API freeze. A release that changes the wire protocol requires rebuilding your integration against the new schema; mismatched client and server are rejected at connection time with `5004 ProtocolVersionMismatch`.

## Versions

- **[v0.7.0](https://docs.tryllengine.com/v0.8/release-notes/v0.7.0.html)**

  Dialog you can put in your save file, one session language for speech, content and prompts, per-model RAM/VRAM placement with engines taken from the catalog, pause timeouts, the Orpheus and Maya1 voices with natural-language voice direction, measured tool-call model rankings, live server RAM/VRAM in the editor, and intent classification from described labels instead of a labelled corpus.

- **[v0.6.0](https://docs.tryllengine.com/v0.8/release-notes/v0.6.0.html)**

  Seeing inside a turn: the shared turn inspector, Agent Log, and Dialog Lab. Plus the manual GPU throttle and background agents, hybrid retrieval (BM25 + RRF), the model-free `Branch` node, and output artifact filtering.

- **[v0.5.0](https://docs.tryllengine.com/v0.8/release-notes/v0.5.0.html)**

  Composable, game-driven turns: the per-turn slot blackboard, agent variables, model-visible tool results, GBNF grammar constraints, dialog-history editing, KV-cache control, and the Unreal graph editor.

- **[v0.4.0](https://docs.tryllengine.com/v0.8/release-notes/v0.4.0.html)**

  Mandatory one-shot `CreateSession`, mid-turn cancel / pause / resume, chat-template-driven tool calling, voice cloning and multilingual TTS.

- **[v0.3.0](https://docs.tryllengine.com/v0.8/release-notes/v0.3.0.html)**

  Server-side voice output (TTS), the Unity workflow graph editor and Model Manager, the bundled server, and file-path storage.

## How to read a version page

- **Highlights** — the marquee capabilities, each linking to the guide or reference page that covers it.
- **Themed sections** — every user-visible change, grouped by area.
- **Breaking changes and behavior changes** — what breaks and how to migrate. Always read this before upgrading.
- **New error codes** — codes that first appear in that release. The full catalog lives in [Error Codes](https://docs.tryllengine.com/v0.8/reference/error-codes.html).
