# Telemetry

Tryll ships with usage telemetry **disabled by default**. This page explains what is collected, where it goes, why, and how to control it — including what you must do before shipping a game to players.

______________________________________________________________________

## Why we collect telemetry

Telemetry is collected exclusively to improve the Tryll engine. Understanding which models are used, how fast inference runs on real hardware, and where errors occur helps us prioritise fixes and optimisations. We do not use it for advertising or share it with third parties.

______________________________________________________________________

## Where data is sent

Events are sent to **[PostHog](https://posthog.com/) (EU region)**, `eu.i.posthog.com`. PostHog is an open-source product analytics platform. Data is stored in the EU.

______________________________________________________________________

## What is collected

All telemetry is emitted by the **Tryll server process**. Nothing is collected by client libraries (C++, Python, Unity, Unreal) themselves.

### Hardware fingerprint — once at startup

A stable, anonymous machine identifier derived from the MAC address and a per-install UUID (SHA-256 hash, hex-encoded — no raw MAC or UUID is ever transmitted). Alongside it: OS version, CPU model and core count, total RAM, GPU names and VRAM, and the active GPU backend (Vulkan / CUDA / ROCm / CPU). This is sent once when the server starts.

### Session events

A session is the logical session created by `CreateSessionRequest`. The following lifecycle events are recorded:

- Session started and session configured (includes the `game_name` identifier your integration sets on `CreateSessionRequest`)
- Session ended (with close reason)
- Agent created and agent destroyed (includes a content hash of the graph, default model name, workload, and KV-cache initialization mode)
- Out-of-turn `ChangeAgentParam` / `UpdateAgentVariables` (field/variable names only — never values)

### Turn performance metrics

For every completed inference turn:

- Status (`success` / `error` / `cancelled`)
- Wall time and time-to-first-token (milliseconds)
- Token count
- Per-node timing, exit route, and turn-relative timestamps
- In-turn control-plane timeline (`events`: pause/resume, param changes, variable updates during pause gaps)
- Error records if the turn failed

### Model download events

When the server downloads a model from HuggingFace: success/failure, duration, and bytes transferred.

### Conversation content — opt-in only

Slot text and tool-call argument blobs are **not included by default**. They are only recorded when `telemetry.include_user_content` is explicitly set to `true` in `server-config.json`. See [Controlling telemetry](#controlling-telemetry) below.

______________________________________________________________________

## Scope — developer tooling, not end-user analytics

The Tryll server is a **developer-side integration tool** that runs on your machine or studio infrastructure. It is not intended to ship as part of a deployed game executable that end users run directly.

**If you are building a game that ships to players, you must remove or disable the PostHog sink before your release build.** See [Controlling telemetry](#controlling-telemetry) below.

______________________________________________________________________

## Controlling telemetry

The `"telemetry"` block in `server-config.json` controls all telemetry behaviour. See [Server Configuration](https://docs.tryllengine.com/reference/server-configuration.html) for the full field reference.

### Turn off all telemetry

Set `"enabled": false` to disable telemetry completely. No fingerprint is captured, no events are queued, and there is zero hot-path cost:

```
"telemetry": {
    "enabled": false
}
```

### Remove the PostHog sink for a shipping build

Remove the `"sinks"` entry (or set `"enabled": false` on the sink) to stop sending data to PostHog without disabling the telemetry infrastructure entirely (useful if you plan to add your own sink later):

```
"telemetry": {
    "enabled": true,
    "include_user_content": false,
    "sinks": []
}
```

### Keep telemetry on but exclude conversation content

Leave telemetry enabled and set `"include_user_content": false` (this is also the default). Performance metrics and lifecycle events are still recorded; no human messages or character replies are ever transmitted:

```
"telemetry": {
    "enabled": true,
    "include_user_content": false,
    "sinks": [
        {
            "kind": "posthog",
            "enabled": true,
            "endpoint": "https://eu.i.posthog.com",
            "project_api_key": "phc_YOUR_KEY"
        }
    ]
}
```

### Suppress telemetry for a process run (CLI override)

For QA, CI pipelines, and internal tooling that should not pollute production analytics, pass `--disable-telemetry` when starting the server:

```
tryll_server.exe --disable-telemetry
```

This overrides `"enabled": true` in `server-config.json` for the lifetime of that process — no sinks are constructed and no events are emitted for any session.

The **qa-and-eval** pipeline passes `--disable-telemetry` to the server automatically when it spawns in managed mode. To allow telemetry through, pass `--enable-telemetry` on the QA CLI:

```
python -m tryll_qa.main --scenario scenarios.npc_v1 --enable-telemetry
```

______________________________________________________________________

## Related

- [Server Configuration](https://docs.tryllengine.com/reference/server-configuration.html) — full `telemetry.*` field reference
- [How to run the Tryll server](https://docs.tryllengine.com/how-to/run-the-tryll-server.html)
