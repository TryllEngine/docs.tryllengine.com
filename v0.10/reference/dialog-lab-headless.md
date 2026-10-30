# Dialog Lab headless run

The run config, the summary file, and the exit codes of a headless Dialog Lab run. Unity and Unreal read the same config and write the same summary, byte for byte. For how to use them, see [Run the Dialog Lab in CI](https://docs.tryllengine.com/v0.10/how-to/run-dialog-lab-in-ci.html).

| Engine | Entry point                                                                                                       |
| ------ | ----------------------------------------------------------------------------------------------------------------- |
| Unity  | `-executeMethod Tryll.Client.Editor.TryllDialogLabBatch.Run -tryllLabConfig <path>` (batch mode, without `-quit`) |
| Unreal | `-run=TryllDialogLab -tryllLabConfig=<path>` (`UnrealEditor-Cmd`)                                                 |

______________________________________________________________________

## Run config

A JSON object, `format` 1. An unknown key is an error, so a typo fails the run instead of being ignored.

| Key                               | Type               | Default                | Meaning                                                                                                                                                  |
| --------------------------------- | ------------------ | ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `format`                          | number             | required               | Must be `1`.                                                                                                                                             |
| `agent.workflow`                  | string             | required               | Workflow asset path (`Assets/...` in Unity, `/Game/...` in Unreal).                                                                                      |
| `agent.name`                      | string             | the asset's name       | Agent and variant label in the results.                                                                                                                  |
| `agent.variables`                 | object             | none                   | Initial values for the workflow's declared variables, by name.                                                                                           |
| `agent.maintain_dialogue_history` | bool               | `true`                 | `false` runs a stateless agent; scripts with seeded turns are then refused.                                                                              |
| `agent.model`                     | string             | workflow default       | Model override, as the Lab's per-variant **Model**.                                                                                                      |
| `dialogues`                       | string             | required               | A Dialog Script or Dialog Suite asset path.                                                                                                              |
| `seeds`                           | whole number ≥ 1   | required               | Runs per dialogue, each with its own seed.                                                                                                               |
| `base_seed`                       | whole number ≥ 1   | `1`                    | First seed; the others follow as in the Lab.                                                                                                             |
| `output_dir`                      | string             | `Build/DialogLab`      | Where results are written. A relative path resolves against the project folder.                                                                          |
| `turn_timeout_s`                  | number ≥ 0         | `120`                  | Seconds a turn may take before the run stops with exit code `2`. `0` means no limit.                                                                     |
| `settings`                        | string             | the project's only one | Unity: a `TryllRuntimeSettings` asset path. Unreal ignores it and uses the project settings.                                                             |
| `thresholds.turns_passed`         | rate 0–1 or `null` | none                   | Generated turns whose checks all passed.                                                                                                                 |
| `thresholds.checks_passed`        | rate 0–1 or `null` | none                   | Authored checks that passed.                                                                                                                             |
| `thresholds.clean_dialogues`      | rate 0–1 or `null` | none                   | Dialogue × seed runs with no failed turn.                                                                                                                |
| `thresholds.use_lower_bound`      | bool               | `false`                | Gate on the lower end of the 95% interval across seeds instead of the rate. Needs `seeds` of 5 or more (intervals start there); fewer is a config error. |

A value in `agent.variables` must match the declared type: a whole number for Int, a number for Float, a string, `true`/`false`, or an array of strings for StringSet. A name the workflow does not declare is an error.

Live tools cannot run headless. Tool calls use the script's canned `ToolResults` only.

______________________________________________________________________

## Outputs

The run writes into `output_dir`:

| File                    | Content                                                                                 |
| ----------------------- | --------------------------------------------------------------------------------------- |
| `<run_id>.jsonl`        | The run in the Lab's export format (format 4). Import it in the Lab to read every turn. |
| `<run_id>.summary.json` | The verdict, described below.                                                           |
| `failed.summary.json`   | Written instead when the run could not start (for example, a missing asset).            |

### Summary JSON

```
{"format":1,"status":"completed","exit_code":1,"run_id":"…",
 "stats":{ … },
 "thresholds":[{"metric":"clean_dialogues","required":0.7,"actual":0.625,
                "basis":"point","passed":false}],
 "issues":[]}
```

| Key          | Meaning                                                                                                                                                                                                                                                |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `status`     | `completed`, `failed`, or `stopped`.                                                                                                                                                                                                                   |
| `exit_code`  | The process exit code.                                                                                                                                                                                                                                 |
| `stats`      | The same statistics block as the JSONL header: per-variant `turns_passed`, `checks_passed`, `clean_dialogues` (each `passed`, `total`, `rate`, and `ci95` from 5 seeds), `errors`, `not_run`, and word-count and time-to-first-token medians and p90s. |
| `thresholds` | One entry per threshold set: the required rate, the `actual` value compared, its `basis` (`point` or `lower_bound`), and whether it `passed`.                                                                                                          |
| `issues`     | Why the run failed, or problems it ran into.                                                                                                                                                                                                           |

Numbers are written with at most four decimals.

______________________________________________________________________

## Exit codes

| Code | Meaning                                                                                                                                                                                                                                                        |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `0`  | The run completed and every threshold was met.                                                                                                                                                                                                                 |
| `1`  | The run completed and at least one threshold was missed.                                                                                                                                                                                                       |
| `2`  | The run failed: config error, missing asset, validation error, server or session failure, an agent that could not be created, a turn timeout, or any generated turn that ended in an error or did not run. Thresholds are not evaluated on incomplete results. |
