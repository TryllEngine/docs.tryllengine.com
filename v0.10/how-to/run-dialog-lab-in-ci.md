# Run the Dialog Lab in CI

Run a [Dialog Lab](https://docs.tryllengine.com/v0.10/how-to/compare-dialog-variants.html) script or suite from the command line, with no editor window, and fail the build when an NPC gets worse. A headless run uses one agent and N seeds. It writes the same JSONL file the Lab exports, plus a short summary, and exits with a code your CI system can gate on.

The file formats and every config key are listed in [Reference: Dialog Lab headless run](https://docs.tryllengine.com/v0.10/reference/dialog-lab-headless.html).

Prerequisites

- A Dialog Script or Dialog Suite asset. See [Compare dialog variants](https://docs.tryllengine.com/v0.10/how-to/compare-dialog-variants.html#run-several-dialogues-at-once).
- A workflow asset for the agent under test.
- A machine that can run the model: a GPU runner, with the models already downloaded (see [Provision models](#provision-models)).

______________________________________________________________________

## Write the run config

Save a JSON file next to your project, for example `ci/lab/bartender.json`:

```
{
  "format": 1,
  "agent": {
    "workflow": "Assets/Workflows/TavernBartender.asset",
    "variables": { "player_name": "Ada" }
  },
  "dialogues": "Assets/Workflows/Tavern_Suite.asset",
  "seeds": 8,
  "output_dir": "Build/DialogLab",
  "thresholds": { "turns_passed": 0.9, "clean_dialogues": 0.7 }
}
```

- **`agent.workflow`** is a workflow asset, not a scene object. Use the asset path: `Assets/...` in Unity, `/Game/...` in Unreal.
- **`dialogues`** is a Dialog Script or a Dialog Suite.
- **`seeds`** is how many times every dialogue runs, each with its own seed.
- **`thresholds`** are pass rates from 0 to 1. Leave one out to report the number without gating on it.

A headless run has one agent. To compare two versions of an NPC, run twice with two configs and compare the two summaries.

______________________________________________________________________

## Run it

```
Unity.exe -batchmode -nographics -projectPath <project> ^
  -executeMethod Tryll.Client.Editor.TryllDialogLabBatch.Run ^
  -tryllLabConfig ci/lab/bartender.json -logFile -
```

Do **not** pass `-quit`. The client needs the editor to keep running while the turns play, and the run ends the editor itself when it is done.

```
UnrealEditor-Cmd.exe <project>.uproject -run=TryllDialogLab ^
  -tryllLabConfig=ci/lab/bartender.json -nullrhi -unattended -nopause
```

The log prints one line per threshold and a verdict:

```
[TryllDialogLab] turns_passed: 0.9375 >= 0.9 (point) PASS
[TryllDialogLab] clean_dialogues: 0.625 < 0.7 (point) FAIL
[TryllDialogLab] Dialog Lab: FAIL, 1 threshold missed (exit 1)
```

| Exit code | Meaning                                                                                                                                                                                                                   |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `0`       | Every threshold met (or no thresholds set).                                                                                                                                                                               |
| `1`       | At least one threshold missed: the NPC got worse.                                                                                                                                                                         |
| `2`       | The run itself failed: bad config, server or model problem, an agent that could not be created, or a turn that timed out. Even one generated turn that errored or did not run counts, because the results are incomplete. |

Keep `1` and `2` apart in your pipeline. A `2` means the test rig is broken, not that the NPC changed.

The editor can also exit with its own code before the run starts. For example, Unity exits `1` when the project is already open in another editor or does not compile. Treat an exit `1` as a missed threshold only when the log has the `Dialog Lab: FAIL` verdict line; without it, the run never happened, so handle it like a `2`.

The results land in `output_dir` as `<run_id>.jsonl` and `<run_id>.summary.json`. Import the JSONL in the Lab (**Import**) to read every answer in the Grid and Statistics views.

______________________________________________________________________

## Provision models

A headless run never downloads models. A missing model makes the run fail with exit code `2`. Download the models the workflow uses before the run, on the runner, and cache them between jobs.

______________________________________________________________________

## Set thresholds that hold

- **Run it nightly, not on every pull request.** Eight seeds across a fifteen-turn suite is 120 model turns per agent.
- **Gate on rates, never on text.** A fixed seed reproduces only on the same server build and hardware.
- **Watch a few runs before you set a threshold.** At 8 seeds the interval on clean dialogues is often ±15 points, so a threshold set too close to the current rate fails at random.
- **`use_lower_bound: true`** gates on the lower end of the 95% interval instead of the rate. That is stricter, so it fails only when a drop is clear. Intervals need at least 5 seeds, so a config that sets it with fewer is refused before the run.
- **`turn_timeout_s`** (default 120) stops the run with exit code `2` when a turn hangs, instead of hanging the job.

______________________________________________________________________

## Related

- [Compare dialog variants in the Lab](https://docs.tryllengine.com/v0.10/how-to/compare-dialog-variants.html)
- [Reference: Dialog Lab headless run](https://docs.tryllengine.com/v0.10/reference/dialog-lab-headless.html)
- [Test tool calls in Chat and the Dialog Lab](https://docs.tryllengine.com/v0.10/how-to/test-tool-calls-in-editor.html)
