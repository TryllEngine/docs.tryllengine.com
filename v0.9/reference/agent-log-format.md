# Agent Log format

**Export** in the [Agent Log](https://docs.tryllengine.com/v0.9/how-to/use-agent-log.html) writes one JSON document. The format identifier is `tryll.agent_log` and `format_version` is `1`. Readers should ignore keys they do not know. There is no importer.

The file can contain player input, prompts, lore, tool arguments, and diagnostics. It is a bug report from this editor's connection, not a redacted transcript.

## Document

Key order is fixed: `format`, `format_version`, `exported_at`, `exporter`, `protocol`, `server`, `capture`, `agents`, `events`, `turns`.

| Key        | Contents                                                                                                                                                                                                                                           |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `exporter` | `unity` or `unreal`, the editor version, the plugin version, the project name, and the OS.                                                                                                                                                         |
| `protocol` | Wire `version` and the codegen fingerprint that rendered the payloads.                                                                                                                                                                             |
| `server`   | Version, run id, build config, and mode, when the connection hello carried them.                                                                                                                                                                   |
| `capture`  | `capturing`, `paused`, `discarded`, or `idle`; retention limits; how many turns and observations were evicted; the oldest retained timestamp.                                                                                                      |
| `agents`   | One row per agent that appeared, with its label, whether diagnostics were on, and `created_seq` / `destroyed_seq`. A null `created_seq` means the agent already existed when capture started.                                                      |
| `events`   | The journal, after export coalescing.                                                                                                                                                                                                              |
| `turns`    | An index. `duration_ms` and `ttft_ms` come from turn diagnostics when that JSON is present. Otherwise they are measured on the client and the row has `"timing_source": "client"`. A turn still in progress has `complete: false` and no `status`. |

A field that does not apply is left out. Ids are never written as `0`.

## Events

| Field                | Meaning                                                                                                                                               |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `seq`                | Journal order. Unique in the file. A coalesced run keeps the first event's `seq`.                                                                     |
| `t_ms`               | Milliseconds since the oldest retained event.                                                                                                         |
| `t_end_ms`, `chunks` | Only on a merged run.                                                                                                                                 |
| `dir`                | `out` or `in`. Omitted for local events.                                                                                                              |
| `kind`               | `request`, `reply`, `push`, `error`, `connection`, `local_error`, or `marker`.                                                                        |
| `type`               | The message or local-event name.                                                                                                                      |
| `turn`               | `epoch:agent:request`.                                                                                                                                |
| `payload`            | The message body rendered from the schema. Bulk fields such as PCM are `{"omitted": true, "count": N, "bytes": B}` and the bytes are not in the file. |

`arguments_json` stays a string. The tool-call panel in the window is what pretty-prints it.

Runs merge only when the schema marks the message for coalescing, and only across events of the same type, the same turn (or the same request or voice stream), and the same node name. A node-name change ends the run. A non-delta answer replaces the text instead of appending.

## Checking a file

From the repo:

```
python tools/agent_log_check.py path/to/TryllAgentLog.json
```

The command checks the envelope and prints agents, turns, errors, and capture gaps. `python tools/agent_log_check.py --compare a.json b.json` compares two exports after dropping timestamps, the exporter block, and renumbering ids.
