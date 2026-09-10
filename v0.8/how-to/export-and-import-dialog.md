# Export and Import Dialog

Capture an agent's **completed** dialog as an owned snapshot, store it in **your** save format, and restore it later. Tryll does not write save files, own slots, or define the outer game container — it only returns a `DialogSnapshot` and a canonical FlatBuffers encoding (file identifier `TDLG`).

Prerequisites

- An agent created and **idle** — no turn running, not paused, and no KV-cache lifecycle operation in flight (same strict-idle rule as [Seed and edit dialog history](https://docs.tryllengine.com/v0.8/how-to/seed-and-edit-dialog-history.html)).
- Import **replaces** the whole dialog. It never appends. A failed import leaves the existing dialog unchanged.

This snapshot is **dialog only**. Agent variables, graph authoring, game progression, UI transcripts, and KV-cache bytes are outside it. After a successful import, call `PrefillKvCache` yourself if you want the prefix warmed — import never prefills.

Encoded snapshots that would overflow the 1 MiB frame cap fail with `DialogSnapshotTooLarge` (`3021`). Unity and Unreal reject the import locally when the encoded snapshot cannot fit in a frame — the connection and the rest of the session stay up. Tryll never truncates.

______________________________________________________________________

## Direct canonical binary

Serialize the snapshot to `TDLG` bytes and write them wherever the game already stores saves:

```
using Tryll.Client;

agent.ExportDialog((snapshot, error) =>
{
    if (!error.IsOk) return;
    var bytes = snapshot.ToFlatBufferBytes();
    File.WriteAllBytes(savePath, bytes); // game-owned path
});

var loaded = TryllDialogSnapshot.FromFlatBufferBytes(File.ReadAllBytes(savePath));
agent.ImportDialog(loaded, TryllDialogCompatibility.Strict, (result, error) => { });
```

```
Agent->ExportDialog(
    [SavePath](const FTryllDialogSnapshot& Snapshot, const FTryllError& Error)
    {
        if (!Error.IsOk()) return;
        TArray<uint8> Bytes;
        FTryllError EncodeError;
        if (!Snapshot.ToFlatBufferBytes(Bytes, EncodeError)) return;
        FFileHelper::SaveArrayToFile(Bytes, *SavePath);
    });

TArray<uint8> Bytes;
FFileHelper::LoadFileToArray(Bytes, *SavePath);
FTryllDialogSnapshot Loaded;
FTryllError DecodeError;
FTryllDialogSnapshot::FromFlatBufferBytes(Bytes, Loaded, DecodeError);
Agent->ImportDialog(Loaded);
```

```
auto snapshot = agent.ExportDialog();
auto bytes = Tryll::Client::EncodeFlatBuffer(snapshot);
WriteSaveFile(path, bytes); // game-owned

auto loaded = Tryll::Client::DecodeFlatBuffer(ReadSaveFile(path));
agent.ImportDialog(loaded);
```

```
from tryll_client.dialog_snapshot import decode_flatbuffer, encode_flatbuffer

snapshot = agent.export_dialog()
Path(save_path).write_bytes(encode_flatbuffer(snapshot))

loaded = decode_flatbuffer(Path(save_path).read_bytes())
agent.import_dialog(loaded)
```

______________________________________________________________________

## Wrap the structure in your save object

Keep the idiomatic snapshot next to the rest of the game save — variables, quest flags, UI transcript — and encode only if you want a compact blob:

```
public class GameSave
{
    public TryllDialogSnapshot Dialog;
    public Dictionary<string, string> Variables;
    public List<string> UiTranscript;
}

agent.ExportDialog((snapshot, error) =>
{
    if (!error.IsOk) return;
    var save = new GameSave { Dialog = snapshot, /* ... */ };
    File.WriteAllText(savePath, JsonUtility.ToJson(save));
});
```

```
snapshot = agent.export_dialog()
save = {
    "dialog": snapshot,  # or encode_flatbuffer(snapshot)
    "variables": agent.variables.get_all(),
    "ui_transcript": ui_lines,
}
# serialize with the game's own format
agent.import_dialog(save["dialog"])
```

`compatibility=Strict` (default) rejects a snapshot whose graph signature does not match the agent's create-time graph (`DialogGraphMismatch` 3020). `AllowGraphMismatch` restores the interactions, remaps compatible projection windows, and returns warnings.

The Murder Mystery Demo is one game-owned wrapper of this pattern: a versioned JSON envelope under `Application.persistentDataPath` that stores Base64 canonical `TDLG` bytes per character (and the note-taker), plus agent variables, disclosure, the client transcript, notes, and investigation progress. Tryll still does not own that file or its schema.

For the complete save sequence, multi-agent restore ordering, atomic slot writes, variable handling, and KV-cache warm-up, see [Save and Load Agent State](https://docs.tryllengine.com/v0.8/how-to/save-and-load-agent-state.html).

Related: [Seed and edit dialog history](https://docs.tryllengine.com/v0.8/how-to/seed-and-edit-dialog-history.html) (lossy authoring verbs — not a substitute for export/import).
