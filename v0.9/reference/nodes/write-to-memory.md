# WriteToMemory Node

**`NodeType = WriteToMemory` (`NodeParams` union discriminator 15)**

Experimental mutable-KB write. Upserts the current input slot into a session-scoped named store. Place a Retrieve on the same store **before** this node in the same turn if the model should see prior memory — a write does not refresh already-projected knowledge.

An empty input writes nothing and takes `written`. Only a rejected write (overflow, schema mismatch, missing store) takes `failed`.

## Parameters

| Param          | Type | Default | Range | Structural | Localizable | Description                                                                                                                |
| -------------- | ---- | ------- | ----- | ---------- | ----------- | -------------------------------------------------------------------------------------------------------------------------- |
| `storage_name` | str  | empty   | —     | ✓          | —           | Session-scoped store name. May contain var.\* templates resolved at agent creation. Structural.                            |
| `schema`       | str  | empty   | —     | ✓          | —           | Catalog schema name. Structural.                                                                                           |
| `input`        | str  | empty   | —     | ✓          | —           | Slot whose text is written. Empty = "user_message".                                                                        |
| `id`           | str  | empty   | —     | —          | —           | Record id, optionally templated with var.\* and resolved per turn. Empty asks the store to assign an insertion-ordered id. |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit      | Param field    | Description                                                        |
| --------- | -------------- | ------------------------------------------------------------------ |
| `written` | `written_exit` | Exit taken after a successful write or a no-op empty input.        |
| `failed`  | `failed_exit`  | Exit taken when the write is rejected (overflow, schema mismatch). |

## Schema picker (Unity)

`schema` is picked from the bundled `data/schemas.json` catalog (same `[TryllSchema]` drawer as Retrieve). `(none)` is offered; unknown values stay visible as **Unregistered**. Server validation is authoritative.

## Exit routes

| Exit      | Condition                                                   |
| --------- | ----------------------------------------------------------- |
| `written` | The upsert was accepted, or the input was empty (no-op).    |
| `failed`  | The upsert was rejected or the per-turn id template failed. |

## First-use cost

The first write to a new store embeds the record inline. Loading a GameMemory snapshot with `embed_timing=Now` batch-embeds every imported store before the response; `Lazy` installs text immediately and embeds on the first dense or hybrid search.
