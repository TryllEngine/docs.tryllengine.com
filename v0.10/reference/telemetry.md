# Telemetry

Your shipped game sends no telemetry

Tryll collects telemetry only from the editor, while you make your game. The server that ships with your game is built without telemetry code. It cannot send anything, and no setting turns it on. See [In your shipped game](#in-your-shipped-game).

The editor server sends a content-free usage report so we can see which models run, how fast they are on real hardware, and where agents fail. That report is the **`basic`** level, and it is the default. Two higher levels, **`turns`** and **`full`**, stay off until you turn them on for a debugging session.

______________________________________________________________________

## In your shipped game

A packaged game runs the player server (Client Prod). That server is built without telemetry, so your players send nothing to Tryll:

- There is no telemetry code in it, not a setting that is off.
- A player config with `telemetry.enabled` set to `true` fails startup instead of sending anything.
- The player build's server log says so. Look for this line:

```
[Telemetry] telemetry=compiled-out
```

Everything below is about the editor.

In the editor, choose the level under **Project Settings → Tryll Client → Telemetry Level** (Unreal: **Project Settings → Plugins → Tryll Client**). The choices are `basic`, `turns`, and `full`. `basic` is the default.

______________________________________________________________________

## Telemetry and the licence

Telemetry is part of the Tryll Runtime Licence. When you accept the licence, you agree to telemetry. There is no separate choice. In the editor, telemetry is always on.

You accept the licence in the **Tryll Welcome** window (Unity: **Window → Tryll → Welcome**; Unreal: **Window → Tryll → Tryll Welcome**). The editor starts the server only after you accept. **Telemetry** opens this page. **Read the licence** shows the full text from the plugin's `LICENSE.md`. The same text is on the [License Agreement](https://docs.tryllengine.com/v0.10/reference/legal/licence.html) page.

If you click **Withdraw**, the editor stops its server and does not start it again until you accept the licence again.

______________________________________________________________________

## Detail levels

| Level       | When it is used                                                                                     | What we receive                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ----------- | --------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`basic`** | **Default.** The editor always sends at least this.                                                 | An anonymous install id, OS, CPU, RAM, and GPU. Session start and end, including the level that is in effect. When an agent is created, a structural summary of its graph: node types, how they connect, and model ids from the shipped catalog (a model you added yourself is recorded as `custom` plus its engine). When the agent is destroyed, lifetime counts and approximate duration percentiles. Parameter and variable changes with the names removed. Download success or failure. Below `full`, every download's model id is recorded as `custom`, shipped catalog models included. |
| **`turns`** | Off until you set it. Use it when a bug is about timing, which node types ran, or how a turn ended. | Everything in `basic`, plus one event per turn: status, duration, time to first token, token counts, and pause time. Each node in the turn contributes its index, schema type, and duration. Tool calls and errors contribute counts only.                                                                                                                                                                                                                                                                                                                                                     |
| **`full`**  | Off until you set it. Use it when we ask for a reproduction that needs the actual text.             | Everything in `turns`, plus prompts, model output, slot text, tool names and arguments, node names, variable names, and error text. Download events keep the model id.                                                                                                                                                                                                                                                                                                                                                                                                                         |

`basic` and `turns` contain no conversation text, no authored names, and no free-text errors. Conversation text is included only at `full`.

There is no `off` level in the editor. If `--telemetry-level off`, `telemetry.level` set to `off`, or `telemetry.enabled` set to `false` reaches the editor server, it raises the level to `basic` and logs a warning. A value other than `off`, `basic`, `turns`, or `full` is a startup error.

The old keys `activity_enabled` and `include_user_content` are ignored. Set `level`.

______________________________________________________________________

## Send more detail while debugging

Set **Telemetry Level** in Project Settings:

- **Unity:** Edit → Project Settings → Tryll Client
- **Unreal:** Edit → Project Settings → Plugins → Tryll Client

Choose **Turns** for timings and counts, or **Full** when the report needs prompts, outputs, or error text. Set it back to **Basic** when the debugging session is over.

The editor passes the choice as `--telemetry-level` the next time it launches the server, and that flag overrides `telemetry.level` in `server-config.json`. Exit Play mode and let the managed server exit (it shuts itself down about 60 seconds after the last session), or quit the editor, then start again. The log line `[Telemetry] level=turns` (or `basic`, or `full`) is the level that took effect. The same value is attached to the session-started event.

A server you start yourself from the release takes the same flag:

```
tryll_server.exe --telemetry-level turns
```

______________________________________________________________________

## Where the editor report goes

Events go to [PostHog](https://posthog.com/) in the EU (`eu.i.posthog.com`). The address is built into the editor server. The `sinks` entry in the config only tunes how events are batched. Removing it leaves `basic` telemetry on.

We use the report to improve the engine: which models are in use, how fast inference is on real hardware, and where errors occur. We do not sell it or use it for advertising. We may use aggregated, anonymized data that no longer identifies you or your project for any lawful business purpose, including product analytics and reporting to investors. We keep telemetry for up to 12 months after collection. Section 5 of the [License Agreement](https://docs.tryllengine.com/v0.10/reference/legal/licence.html#s5) and the [Privacy Policy](https://docs.tryllengine.com/v0.10/reference/legal/privacy-policy.html) are the full terms.

The install id is a random identifier stored on the machine. It is not derived from a hardware address. Events from one project are grouped by a project id the editor generates once and saves with the project, so a clone or a teammate checkout stays on the same id. **Game Name** in Project Settings is the human label: it is sent as the session name and as `--project-name`, and it is used for log folders together with the project directory. The telemetry group is the project id.

Client libraries do not collect telemetry themselves. The server process does. The one exception is the server install event below.

______________________________________________________________________

## Server install event

The plugin sends one event, `tryll.server.install`, to the same PostHog project:

- **Unity:** a released package downloads its server once, from the **Welcome** window (**Window → Tryll → Welcome**). The event is sent after each install attempt, from a download or from a zip.
- **Unreal:** the server comes inside the plugin. The event is sent once per plugin version, after you accept the licence.

It carries:

- the result (`ok` or an error code), the server version and variant (`lite` or `full`)
- how the server was installed (`download` or `zip`) and the release channel
- the download size and duration (`0` in Unreal, where nothing is downloaded)
- the engine (`unity` or `unreal`) and the editor version
- a random install id, kept for your user account on the computer and never derived from hardware

It carries no paths, names, prompts or project content. Like the editor report above, it is part of the telemetry you agree to with the licence.

______________________________________________________________________

## Who sends what

| Where you run                                                                                     | What is sent                                                                                                                         |
| ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| **Editor server** (the Unity or Unreal editor, or the release server started by hand; Client Dev) | The Project Settings level, default `basic`. The floor is `basic`.                                                                   |
| **Packaged player** (Client Prod)                                                                 | Nothing. The log line is `[Telemetry] telemetry=compiled-out`. A player config with `telemetry.enabled` set to `true` fails startup. |

______________________________________________________________________

## Related

- [Server Configuration](https://docs.tryllengine.com/v0.10/reference/server-configuration.html) — `telemetry.level` and the batch fields
- [How to run the Tryll server](https://docs.tryllengine.com/v0.10/how-to/run-the-tryll-server.html)
- [Auto-launch the server](https://docs.tryllengine.com/v0.10/how-to/auto-launch-server.html)
