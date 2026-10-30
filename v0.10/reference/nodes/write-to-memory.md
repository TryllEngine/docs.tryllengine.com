# WriteToMemory Node

**`NodeType = WriteToMemory` (`NodeParams` union discriminator 15)**

Experimental mutable-KB write. Upserts the current input slot into a session-scoped named store. Place a Retrieve on the same store **before** this node in the same turn if the model should see prior memory — a write does not refresh already-projected knowledge.

A blank `input` reads the player's message, so it stores the player line. The node writes nothing and takes `written` only when that resolved text is empty. Only a rejected write (for example the store is full) or a failed per-turn id template takes `failed`. A store that does not exist yet is created when the agent is created. Agent creation fails instead when the store name or schema is invalid or unknown, or the store already exists under a different schema.

## Parameters

### Input

| Param   | Type | Default | Range | Structural | Localizable | Description                                                                                    |
| ------- | ---- | ------- | ----- | ---------- | ----------- | ---------------------------------------------------------------------------------------------- |
| `input` | str  | empty   | —     | ✓          | —           | Text to store: the player's message or an earlier node's output. Empty = the player's message. |

### Storage

| Param          | Type | Default | Range | Structural | Localizable | Description                                                                                                                       |
| -------------- | ---- | ------- | ----- | ---------- | ----------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `storage_name` | str  | empty   | —     | ✓          | —           | Memory store to write to, by name; it lasts for the session. May include agent variables (var.\*).                                |
| `schema`       | str  | empty   | —     | ✓          | —           | Record schema of the store. Nodes that read the same store must use the same schema.                                              |
| `id`           | str  | empty   | —     | —          | —           | Record id; writing an existing id replaces that record. May include agent variables (var.\*). Empty = add a new record each time. |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit      | Param field    | Description                                                                                                 |
| --------- | -------------- | ----------------------------------------------------------------------------------------------------------- |
| `written` | `written_exit` | Node to run after the write, or when the text to store is empty (nothing is written). Empty = end the turn. |
| `failed`  | `failed_exit`  | Node to run when the store rejects the write (for example, the store is full). Empty = end the turn.        |

## Schema picker (Unity)

`schema` is picked from the bundled `data/schemas.json` catalog (same `[TryllSchema]` drawer as Retrieve). `(none)` is offered; unknown values stay visible as **Unregistered**. Server validation is authoritative.

## Exit routes

| Exit      | Condition                                                                 |
| --------- | ------------------------------------------------------------------------- |
| `written` | The upsert was accepted, or the resolved text to store was empty (no-op). |
| `failed`  | The upsert was rejected or the per-turn id template failed.               |

## First-use cost

The first write to a new store embeds the record inline. Loading a GameMemory snapshot with `embed_timing=Now` batch-embeds every imported store before the response; `Lazy` installs text immediately and embeds on the first dense or hybrid search.

## Related

- [Inspect and Edit Knowledge Stores](https://docs.tryllengine.com/v0.10/how-to/inspect-knowledge-stores.html) — see what this node wrote, and query or edit the store in the editor
