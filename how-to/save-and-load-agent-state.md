# Save and Load Agent State

Add Tryll agents to an existing game-save system without losing their conversation context.

Tryll provides the dialog snapshot; your game still owns the save slots, file format, versioning, validation, and non-Tryll state. For the snapshot API and client-specific encoding examples, first read [Export and Import Dialog](https://docs.tryllengine.com/how-to/export-and-import-dialog.html).

Prerequisites

- Agents created and **idle** — no turn running, not paused, and no KV-cache lifecycle operation in flight.
- A **fresh session** on load: recreate agents, then import. Do not import into an agent that is mid-turn.
- Unreal: `ExportDialog` / `ImportDialog` are **C++ only**. The snapshot type is not Blueprint-exposed.

## Decide what belongs in the save

An exported dialog snapshot contains the completed interactions and the projection state needed to continue the conversation faithfully. It does **not** contain:

- agent variables;
- the workflow graph or model files;
- UI transcript rows;
- quest, world, or progression state;
- KV-cache bytes;
- the agent or session ID.

Store those values beside the snapshot in a versioned, game-owned envelope. Use an authored key such as a character ID or workflow asset GUID to identify an agent. Agent and session IDs are runtime handles and change when the game starts again.

For example, a Unity JSON save can store each canonical `TDLG` snapshot as Base64:

```
[Serializable]
public sealed class GameSave
{
    public int formatVersion = 1;
    public string scenarioKey;
    public SavedAgent[] agents;
    public QuestState quests;
}

[Serializable]
public sealed class SavedAgent
{
    public string characterKey;   // Stable authored identity, never AgentId
    public string dialogBase64;   // Canonical TDLG bytes
    public SavedVariable[] variables;
    public TranscriptLine[] transcript;
}
```

The outer schema is yours. Changing it does not require a Tryll protocol change. When a game update changes workflow graphs, bump **your** envelope version (or migrate) so a strict import fails with a useful message instead of `DialogGraphMismatch`.

## Save only at a quiescent point

`ExportDialog` is strict idle-only. It returns `AgentBusy` while:

- a turn is generating or paused;
- a KV-cache lifecycle operation is running; or
- another dialog or lifecycle request is in flight.

Lock player input first, wait for every turn to complete, and check lifecycle-idle before starting the export. Your game must track whether a sent turn is still in flight; Unity's `agent.IsLifecycleIdle` tracks lifecycle requests and pauses, not the turn itself.

Do not cancel a turn merely to make a save unless that is your game's intended behavior. A paused tool call is still an open turn and must be resumed or cancelled before saving.

If you use [`TryllKvResidencyPolicy`](https://docs.tryllengine.com/how-to/run-many-agents-on-a-small-gpu.html), pause its scheduling around the transaction so a debounce cannot start a prefill or eviction while you wait for idle:

```
policy.SchedulingSuspended = true;   // Unity: still flushes queued sends
try
{
    yield return WaitUntilIdle();
    // export every agent, then write the slot
}
finally
{
    policy.SchedulingSuspended = false;
}
```

On Unreal, do not call `SetActiveAgent` during the wait, and finish any in-flight cache operation before export. Register or enable the policy only after restore has imported dialogs and warmed the agents you want resident.

## Capture each agent

For every persistent agent:

1. Synchronize its variables from the server if the save needs the authoritative values.
1. Capture the typed variable values in your own schema.
1. Export the dialog.
1. Encode the snapshot as canonical `TDLG` bytes.

```
agent.Variables.SyncFromServerAsync(null, syncError =>
{
    if (!syncError.IsOk)
    {
        FailSave(syncError.Message);
        return;
    }

    var savedVariables = CaptureVariables(agent.Variables);

    agent.ExportDialog((snapshot, exportError) =>
    {
        if (!exportError.IsOk)
        {
            FailSave(exportError.Message);
            return;
        }

        var saved = new SavedAgent
        {
            characterKey = characterKey,
            dialogBase64 = Convert.ToBase64String(snapshot.ToFlatBufferBytes()),
            variables = savedVariables,
            transcript = CaptureTranscript(),
        };
        ContinueSave(saved);
    });
});
```

```
Agent->GetVariables().SyncFromServerAsync({}, [Agent, CharacterKey](const FTryllError& SyncError)
{
    if (!SyncError.IsOk()) { FailSave(SyncError.Message); return; }

    Agent->ExportDialog(
        [CharacterKey](const FTryllDialogSnapshot& Snapshot, const FTryllError& Error)
        {
            if (!Error.IsOk()) { FailSave(Error.Message); return; }

            TArray<uint8> Bytes;
            FTryllError EncodeError;
            if (!Snapshot.ToFlatBufferBytes(Bytes, EncodeError))
            {
                FailSave(EncodeError.Message);
                return;
            }

            FSavedAgent Saved;
            Saved.CharacterKey = CharacterKey;
            Saved.DialogBase64 = FBase64::Encode(Bytes);
            Saved.Variables = CaptureVariables();
            ContinueSave(Saved);
        });
});
```

```
agent.Variables().SyncFromServer();
auto savedVariables = CaptureVariables(agent.Variables());

auto snapshot = agent.ExportDialog();
auto bytes = Tryll::Client::EncodeFlatBuffer(snapshot);

SavedAgent saved;
saved.characterKey = characterKey;
saved.dialogBase64 = Base64Encode(bytes);
saved.variables = std::move(savedVariables);
```

```
from tryll_client.dialog_snapshot import encode_flatbuffer

agent.variables.sync_from_server()
saved = {
    "character_key": character_key,
    "dialog": encode_flatbuffer(agent.export_dialog()),
    "variables": capture_variables(agent.variables),
    "transcript": capture_transcript(),
}
```

`ExportDialog` flushes locally staged variable writes before it sends the snapshot request. The explicit synchronization above is still useful when the save must read back the server's authoritative variable mirror.

Export all required agents before writing the file. If any export fails, keep the previous slot untouched.

## Commit the save atomically

Before replacing a slot:

1. Confirm every expected stable agent key appears exactly once.
1. Confirm every dialog blob decodes (`FromFlatBufferBytes` / `DecodeFlatBuffer` / `decode_flatbuffer`).
1. Serialize the complete envelope to a temporary sibling file.
1. Replace the previous slot only after the temporary write succeeds.

This prevents a crash, disk error, or failed agent export from destroying the last valid save.

## Restore in a fresh session

Restore after the session and agents have been recreated:

1. Read and validate the entire envelope before changing live state.
1. Recreate each agent with the expected workflow graph and variable declarations.
1. Match saved records to agents by stable authored key.
1. Decode and import every dialog with `Strict` compatibility.
1. Restore and flush agent variables.
1. Restore UI, quest, world, and progression state.
1. Optionally prefill the currently active agents' KV caches, then start any residency policy.

A scene or level reload is a reliable all-or-nothing wrapper: validate the slot first, then load into a new session and abandon that session if any import fails.

```
var bytes = Convert.FromBase64String(saved.dialogBase64);
var snapshot = TryllDialogSnapshot.FromFlatBufferBytes(bytes);

agent.ImportDialog(
    snapshot,
    TryllDialogCompatibility.Strict,
    (result, importError) =>
    {
        if (!importError.IsOk)
        {
            FailLoad(importError.Message);
            return;
        }

        RestoreVariables(agent.Variables, saved.variables);
        agent.Variables.FlushIfDirty(flushError =>
        {
            if (!flushError.IsOk)
            {
                FailLoad(flushError.Message);
                return;
            }

            RestoreTranscript(saved.transcript);
            ContinueLoad();
        });
    });
```

```
TArray<uint8> Bytes;
FBase64::Decode(Saved.DialogBase64, Bytes);

FTryllDialogSnapshot Snapshot;
FTryllError DecodeError;
if (!FTryllDialogSnapshot::FromFlatBufferBytes(Bytes, Snapshot, DecodeError))
{
    FailLoad(DecodeError.Message);
    return;
}

Agent->ImportDialog(
    Snapshot,
    ETryllDialogCompatibility::Strict,
    [Agent, Saved](const FTryllDialogImportResult& Result, const FTryllError& Error)
    {
        if (!Error.IsOk()) { FailLoad(Error.Message); return; }

        RestoreVariables(Agent->GetVariables(), Saved.Variables);
        Agent->GetVariables().FlushIfDirty(
            [](const FTryllError& FlushError)
            {
                if (!FlushError.IsOk()) FailLoad(FlushError.Message);
                else ContinueLoad();
            });
    });
```

```
auto bytes = Base64Decode(saved.dialogBase64);
auto snapshot = Tryll::Client::DecodeFlatBuffer(bytes);
auto result = agent.ImportDialog(snapshot); // Strict by default

RestoreVariables(agent.Variables(), saved.variables);
agent.Variables().Flush();
```

```
from tryll_client.dialog_snapshot import decode_flatbuffer

snapshot = decode_flatbuffer(saved["dialog"])
agent.import_dialog(snapshot)  # Strict by default
restore_variables(agent.variables, saved["variables"])
agent.variables.flush_if_dirty()
restore_transcript(saved["transcript"])
```

Import replaces the agent's whole dialog; it never appends. A failed import is atomic for that agent and leaves its current dialog unchanged.

A multi-agent restore is not one transaction

If three agents import successfully and the fourth fails, Tryll does not roll back the first three. Decode and validate every record up front. For strict all-or-nothing loading, restore into a fresh scene/session and abandon that session if any import fails.

## Choose compatibility deliberately

Use `Strict` for normal game loading. It verifies that the saved graph signature matches the recreated agent and returns `DialogGraphMismatch` (`3020`) if it does not.

`AllowGraphMismatch` is a migration tool. It restores compatible interactions, remaps projection windows where possible, and returns warnings. Do not enable it silently for production saves: inspect the warnings and test the changed workflow first.

Version the outer save separately from the Tryll snapshot format. When a game update changes character keys, variable declarations, or workflow graphs, migrate or reject the game save explicitly.

## Warm only the agents that need it

Dialog import invalidates the reusable LLM prefix and never prefills it. The next `SendMessage` restores the context automatically, but its first-token latency can be higher.

During a loading screen, call `PrefillKvCache` for the active character or a small hot set. If a residency policy will manage those agents afterwards, mark a successful restore prefill as already resident so the policy does not repeat it (Unity: `policy.MarkResident(component)`).

```
agent.PrefillKvCache((result, error) =>
{
    if (!error.IsOk)
        Debug.LogWarning($"KV prefill failed: {error.Message}");
    else if (policy != null)
        policy.MarkResident(component);
});
```

```
auto result = agent.PrefillKvCache();
if (!result.applicable) {
    // Graph has no eligible LLM nodes; nothing to warm.
}
```

```
result = agent.prefill_kv_cache()
```

Avoid prefilling every inactive NPC unless the memory and loading-time cost is intentional. See [Manage an Agent's KV Cache](https://docs.tryllengine.com/how-to/manage-agent-kv-cache.html) and [Run many agents on a small GPU](https://docs.tryllengine.com/how-to/run-many-agents-on-a-small-gpu.html).

## Handle expected failures

| Error                                 | Meaning                                                                                                        | Save/load response                                                                                                |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `AgentBusy` (`3004`)                  | Turn, pause, cache operation, or snapshot request is active                                                    | Keep input locked, wait for idle, and retry with a timeout.                                                       |
| `DialogSnapshotInvalid` (`3018`)      | Snapshot data is malformed                                                                                     | Reject the save.                                                                                                  |
| `DialogFormatUnsupported` (`3019`)    | Snapshot format is not supported by this build                                                                 | Reject or migrate the save.                                                                                       |
| `DialogGraphMismatch` (`3020`)        | Strict import found a different workflow graph                                                                 | Load the matching game version or run an explicit migration.                                                      |
| `DialogSnapshotTooLarge` (`3021`)     | Snapshot exceeds the 1 MiB frame cap. Unity/Unreal reject the import on the client so the connection stays up. | Do not truncate the encoded blob; start a new dialog or deliberately reduce retained history before a later save. |
| `DialogUnsupportedComponent` (`3022`) | Snapshot contains a component this build cannot import                                                         | Reject or migrate the save.                                                                                       |

Always bound idle waits and retries. Surface a useful error to the player instead of leaving the save UI spinning indefinitely.

Unity: use `TryllAgentCoroutines`

The Unity plugin ships coroutine wrappers — `TryllAgentCoroutines.ExportDialog`, `ImportDialog`, `SyncVariablesFromServer`, `FlushVariables`, and `PrefillKvCache` — that retry `AgentBusy` at a polite interval and bound every wait with a wall-clock deadline. On a dropped connection they report `Timeout` (`1003`) instead of leaving the coroutine waiting forever.

## Reference implementation

The Murder Mystery Demo implements this pattern for four suspects and a background note-taker:

- `Assets/Scripts/Save/SaveData.cs` — versioned game-owned envelope;
- `Assets/Scripts/Save/SaveService.cs` — validation and atomic slot writes;
- `Assets/Scripts/Save/SaveCoordinator.cs` — quiescent multi-agent export, ordered strict restore, variable restoration, and hot-set prefill.

The demo stores dialog, variables, disclosure state, UI transcripts, notes, and investigation progress together while keeping the canonical Tryll snapshot opaque. Guard-blocked lines stay in the client transcript (marked out-of-character) and are absent from the server snapshot.

## Related

- [Export and Import Dialog](https://docs.tryllengine.com/how-to/export-and-import-dialog.html)
- [Agent Variables](https://docs.tryllengine.com/concepts/agent-variables.html)
- [Manage an Agent's KV Cache](https://docs.tryllengine.com/how-to/manage-agent-kv-cache.html)
- [Run many agents on a small GPU](https://docs.tryllengine.com/how-to/run-many-agents-on-a-small-gpu.html)
- [Seed and Edit Dialog History](https://docs.tryllengine.com/how-to/seed-and-edit-dialog-history.html)
