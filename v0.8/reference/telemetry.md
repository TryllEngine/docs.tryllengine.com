# Telemetry

Tryll collects usage data to improve the engine. What is sent depends on the **deployment mode**. This page explains the lanes, what is collected, where it goes, and how to control it.

______________________________________________________________________

## Modes at a glance

| Mode                           | Usage lane (host + hardware + session started)                                                                                                                                                                                | Activity lane (agents, turns, downloads, …)    |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| **Tryll Dev**                  | Optional. Off unless `telemetry.enabled=true` (the built-in default is `false`; the shipped `server-config.json` sets `true`). `telemetry.enabled=false` or `--disable-telemetry` turns **everything** off, including host ID | Optional                                       |
| **Client Dev** (editor plugin) | **Always on.** Setting `enabled=false` is ignored with a warning                                                                                                                                                              | Gated by `activity_enabled` (default **true**) |
| **Client Prod / Standalone**   | Off                                                                                                                                                                                                                           | Off                                            |

`include_user_content` defaults **true** and applies **only** to the activity lane. The usage lane never carries conversation text.

Until a Client Prod binary ships, packaged players still launch Client Dev — so they currently emit mandatory usage telemetry. Those player builds are **unreleasable**. Do not ship them to end users yet.

For Tryll’s own CI and developers, set `TRYLL_SERVER_MODE=tryll-dev` so the Unity/Unreal plugin launches Tryll Dev with `--disable-telemetry`.

______________________________________________________________________

## Why we collect telemetry

Telemetry is collected exclusively to improve the Tryll engine. Understanding which models are used, how fast inference runs on real hardware, and where errors occur helps us prioritise fixes and optimisations. We do not use it for advertising or share it with third parties.

______________________________________________________________________

## Where data is sent

Events are sent to **[PostHog](https://posthog.com/) (EU region)**, `eu.i.posthog.com`, the one sink listed in the shipped `server-config.json`. PostHog is an open-source product analytics platform. Data is stored in the EU. A config with an empty `telemetry.sinks` list sends nothing.

______________________________________________________________________

## What is collected

All telemetry is emitted by the **Tryll server process**. Nothing is collected by client libraries (C++, Python, Unity, Unreal) themselves.

### Usage lane — host and session start

A stable, anonymous machine identifier: a 128-bit FNV-1a hash over the first non-zero network adapter MAC address followed by a per-install UUID. No raw MAC or UUID is transmitted. Alongside it: OS version, CPU model and core count, total RAM, GPU names and VRAM, and the active GPU backend. Also: `tryll.session.started` with `game_name`.

### Activity lane — after session start

- Session configured / ended
- Agent created and destroyed
- Out-of-turn param / variable changes (names only)
- Turn performance metrics and optional user content
- Model download success / failure

### Conversation content

Slot text and tool-call argument blobs are included in the activity lane when `telemetry.include_user_content` is `true` (the default). Set it to `false` to strip that content. The usage lane is unaffected.

______________________________________________________________________

## Controlling telemetry

The `"telemetry"` block in `server-config.json` controls behaviour. See [Server Configuration](https://docs.tryllengine.com/v0.8/reference/server-configuration.html) for the full field reference.

### Tryll Dev — turn off all telemetry

```
"telemetry": {
    "enabled": false
}
```

Or pass `--disable-telemetry` / set `TRYLL_SERVER_MODE=tryll-dev` for the plugin.

### Client Dev — keep usage, quiet activity

```
"telemetry": {
    "enabled": true,
    "activity_enabled": false,
    "include_user_content": false
}
```

Usage events still fire. Setting `enabled: false` does **not** stop them.

### Suppress telemetry for a process run (CLI override)

For QA, CI, and internal tooling runs that should not pollute production analytics, pass `--disable-telemetry` when starting the server **in Tryll Dev**:

```
tryll_server.exe --mode tryll-dev --disable-telemetry
```

The **qa-and-eval** pipeline passes `--disable-telemetry` automatically when it spawns in managed mode. Pass `--enable-telemetry` on the QA CLI to allow it through.

______________________________________________________________________

## Related

- [Server Configuration](https://docs.tryllengine.com/v0.8/reference/server-configuration.html) — full `telemetry.*` field reference
- [How to run the Tryll server](https://docs.tryllengine.com/v0.8/how-to/run-the-tryll-server.html)
