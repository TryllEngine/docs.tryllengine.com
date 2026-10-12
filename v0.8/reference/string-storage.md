# String Storage

A **string storage** is a session-owned, named container of strings. It exists to back workflow nodes with fixed text content without baking that content into the graph definition at authoring time.

String storages are **not embedded** and **not indexed**. For vector retrieval, see [embedded string storage](https://docs.tryllengine.com/v0.8/reference/embedded-string-storage.html).

## Kinds

Every string storage has a **kind** set at creation time; the kind determines what accessors the server exposes to nodes and what validation rules apply.

| Kind       | Description                                 | Key uniqueness           |
| ---------- | ------------------------------------------- | ------------------------ |
| `List`     | Ordered sequence of plain strings. No keys. | —                        |
| `Map`      | Ordered sequence of `key → value` pairs.    | Each key must be unique. |
| `Multimap` | Ordered sequence of `key → value` pairs.    | Duplicate keys allowed.  |

All three kinds share a **list view** — the server can iterate every value by position regardless of kind. `Map` and `Multimap` additionally expose a **keyed view** (`FindFirst`, `FindAll`) that is used by nodes which need to look up values by a string key.

Internal storage order for Map and Multimap

The server stores `Map` and `Multimap` entries **sorted by key** (lexicographic, stable within each key group so that duplicate-key values appear in their original insertion order). This makes `FindFirst` and `FindAll` O(log n) binary-search lookups, and guarantees that `FindAll` returns every matching value even when the keys were not grouped at creation time. The list view (`GetByIndex`) reflects this sorted order rather than the original construction order.

Current node support

`CannedResponseNode` and `RegexGuardrailNode` use only the **list view** — they iterate all values in order and are not key-aware. Both nodes accept any kind.

[`IntentToInstructionNode`](https://docs.tryllengine.com/v0.8/reference/nodes/intent-to-instruction.html) uses the **keyed view** (`FindFirst`) and **requires `Map` kind**. It is the primary consumer of Map-kind storages today.

Key-aware `CannedResponse` selection (keyed by a runtime intent label) is planned as future work.

## Lifecycle

**The preferred way to supply string data to a node is a relative file path** set directly on the node's `string_storage` parameter. The server loads the file on demand when the agent is created — no explicit `Create` / `Destroy` is required.

Use explicit `CreateStringStorageRequest` / `DestroyStringStorageRequest` only when you need to share the same in-memory storage across multiple agents, or when you want to supply inline strings rather than a file.

| Step               | Frame                                         | Notes                                                                                                                                                                                                                                                                           |
| ------------------ | --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Lazy load**      | *(none)*                                      | Set `string_storage = "relative/path.txt"` on the node param. The server resolves the path against the process storage root at `CreateAgentRequest` time and loads (or reuses) the storage.                                                                                     |
| **Explicit Pin**   | `CreateStringStorageRequest`                  | Pins the named storage so it is not evicted between agent creations. Server responds with `CreateStringStorageResponse`.                                                                                                                                                        |
| **Use**            | *(referenced by node `string_storage` param)* | Scoped to the [session](https://docs.tryllengine.com/v0.8/reference/glossary.html#session). For file-backed storages (lazy or pinned with `file_path`) the param value is the relative file path; for inline storages created with `strings` it is the `name` used at creation. |
| **Explicit Unpin** | `DestroyStringStorageRequest`                 | Demotes a file-backed storage from Pinned to OnDemand (eligible for eviction) or removes a virtual/inline entry. Server responds with `Ack`. Agents that already hold the storage keep it alive — the call only removes the session's named reference.                          |

The session drops every storage it owns on disconnect. See [Lifetime and Ownership → StringStorage](https://docs.tryllengine.com/v0.8/concepts/lifetime-and-ownership.html#stringstorage) for the full ownership model.

## Node param: relative file path (preferred)

Set the node's `string_storage` parameter to a **relative path** under the process storage root:

```
string_storage = "responses/refusal_lines.txt"   // List
string_storage = "intent/instructions.json"       // Map (IntentToInstruction)
```

The path is resolved at `CreateAgentRequest` time against the process storage root, set at server startup (see [Server Configuration → Storage root](https://docs.tryllengine.com/v0.8/reference/server-configuration.html#storage-root) for the resolution chain). There is no per-session root. Extension inference:

| Extension  | Inferred kind |
| ---------- | ------------- |
| `.txt`     | List          |
| `.json`    | Map           |
| `.mm.json` | Multimap      |

If the file is outside the root or does not exist, the server returns `3009 StorageOutsideRoot` or `3010 StorageFileNotFound` on `CreateAgentRequest`. If the file kind does not match what the node requires, it returns `3011 StorageKindMismatch`.

## Creation (explicit Pin)

`CreateStringStorageRequest` carries these fields:

| Field       | Type                     | Description                                                                                                                                |
| ----------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `name`      | string                   | Unique name within the session. Required for inline storages. Ignored when `file_path` is set: a file-backed storage is keyed by its path. |
| `kind`      | `StringStorageKind` enum | `List` (0, default), `Map` (1), or `Multimap` (2).                                                                                         |
| `strings`   | `[string]`               | Inline values. For `List`: the strings themselves. For `Map`/`Multimap`: the values, parallel to `keys`.                                   |
| `keys`      | `[string]`               | Inline keys (Map/Multimap only). Must have the same length as `strings`.                                                                   |
| `file_path` | string                   | Relative path to a storage file under the process storage root. Extension → kind inference applies (`.txt` → List, `.json` → Map).         |

Exactly one content source must be provided:

- `strings` alone → **List** inline (virtual/named registration).
- `strings` + `keys` → **Map** or **Multimap** inline (virtual/named).
- `file_path` → file-based (Pins the loaded storage against eviction).

Old clients that do not set `kind` default to `List` and keep working without any changes.

### File format — List (`.txt`)

UTF-8 text, one value per line. Lines starting with `#` and blank lines are ignored:

```
# Lines starting with # are comments.
# Blank lines are also ignored.

I can't help with that.
I can't assist with that request.
```

### File format — Map / Multimap (`.json`)

A JSON **array** of objects with `id` (key) and `text` (value) fields. This is the same on-disk shape used by knowledge-base records; a `metadata` field is accepted but ignored in this context.

```
[
  {"id": "greet_friendly", "text": "Hello there, traveller!"},
  {"id": "greet_friendly", "text": "Well met!"},
  {"id": "greet_hostile",  "text": "What do you want?"}
]
```

Duplicate `id` values are rejected for `Map` kind and accepted for `Multimap` kind.

Note

Relative paths in `file_path` (and in the node `string_storage` param) are resolved against the **process storage root** — CLI `--storage-root`, config `storage_root`, or the mode default (Client Dev: `<Project>/TryllStorage/`). Protocol v24 removed per-session `storage_data_folder`. Never use absolute paths; they are rejected with `3009 StorageOutsideRoot`.

## Server defaults

If a canned-response or guardrail node is created with **no** `string_storage` param and an empty `inline_strings` list, the server falls back to a file configured in [`server-config.json`](https://docs.tryllengine.com/v0.8/reference/server-configuration.html):

- `default_canned_responses_path` — default responses (List kind).
- `default_guardrail_patterns_path` — default patterns (List kind, regex).

Both files are plain `.txt` (List kind). There are no JSON defaults.

## Resolution order

A canned-response or guardrail node resolves its string source in this order:

1. **Named storage** — `string_storage` param set → look up in the session's storage manager.
1. **Inline strings** — `inline_strings` non-empty → use as-is.
1. **Server default** — load from the configured default file.

## Minimum working examples

```
client.create_string_storage(
    name="refusal_lines",
    strings=[
        "I can't help with that.",
        "That's outside what I can do.",
    ],
)
# Reference "refusal_lines" from a CannedResponse node
# via the `string_storage` param.
```

```
client.CreateStringStorage("refusal_lines", {
    "I can't help with that.",
    "That's outside what I can do.",
});
```

```
auto* Subsystem = GetGameInstance()->GetSubsystem<UTryllSubsystem>();
Subsystem->RequestCreateStringStorage(
    TEXT("refusal_lines"),
    { TEXT("I can't help with that."),
      TEXT("That's outside what I can do.") });
```

```
client.create_keyed_string_storage(
    name="intent_instructions",
    keys=["greet", "farewell"],
    values=["Say hello warmly.", "Say goodbye politely."],
    kind=StringStorageKind.Map,
)
```

```
client.CreateStringStorageKeyed(
    "intent_instructions",
    {"greet", "farewell"},
    {"Say hello warmly.", "Say goodbye politely."},
    ::Tryll::StringStorageKind_Map);
```

```
auto* Subsystem = GetGameInstance()->GetSubsystem<UTryllSubsystem>();
Subsystem->RequestCreateStringStorageKeyed(
    TEXT("intent_instructions"),
    { TEXT("greet"), TEXT("farewell") },
    { TEXT("Say hello warmly."), TEXT("Say goodbye politely.") },
    1 /* Map */);
```

```
await client.RequestCreateKeyedStringStorageAsync(
    "intent_instructions",
    new List<string> { "greet", "farewell" },
    new List<string> { "Say hello warmly.", "Say goodbye politely." },
    Tryll.StringStorageKind.Map);
```

## Errors

| Code                                                                                       | Cause                                                                                                                                           |
| ------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| [`3009`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#3xxx-agent)          | `string_storage` param path escapes the storage root (absolute path or `..` traversal). Fix: use a relative path.                               |
| [`3010`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#3xxx-agent)          | `string_storage` param path resolves inside the root but the file is absent. Fix: place the file under the process storage root.                |
| [`3011`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#3xxx-agent)          | `string_storage` param file kind does not match what the node requires (e.g. `.txt` for `IntentToInstruction` which requires Map).              |
| [`7001`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#7xxx-string-storage) | Name empty, malformed, or reserved.                                                                                                             |
| [`7002`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#7xxx-string-storage) | A storage with this name already exists in the session.                                                                                         |
| [`7003`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#7xxx-string-storage) | Content rejected: unreadable file, empty array, bad format, duplicate key in Map, size mismatch between `keys` and `strings`.                   |
| [`7004`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#7xxx-string-storage) | Inline virtual name shadows an existing real file under the storage root. Choose a name that does not match any filename in the storage folder. |

## Client bindings

|            | List (inline)                                    | List (file)                                           | Map / Multimap (inline)                                          | Map / Multimap (file)                                       |
| ---------- | ------------------------------------------------ | ----------------------------------------------------- | ---------------------------------------------------------------- | ----------------------------------------------------------- |
| **C++**    | `CreateStringStorage(name, strings)`             | `CreateStringStorageFromFile(name, path)`             | `CreateStringStorageKeyed(name, keys, values, kind)`             | `CreateStringStorageFromFile(name, path, kind)`             |
| **Python** | `create_string_storage(name, strings=…)`         | `create_string_storage(name, file_path=…)`            | `create_keyed_string_storage(name, keys, values, kind)`          | `create_string_storage(name, file_path=…, kind=…)`          |
| **Unreal** | `RequestCreateStringStorage(Name, Strings)`      | `RequestCreateStringStorageFromFile(Name, Path)`      | `RequestCreateStringStorageKeyed(Name, Keys, Values, Kind)`      | `RequestCreateStringStorageFromFile(Name, Path, Kind)`      |
| **Unity**  | `RequestCreateStringStorageAsync(name, strings)` | `RequestCreateStringStorageFromFileAsync(name, path)` | `RequestCreateKeyedStringStorageAsync(name, keys, values, kind)` | `RequestCreateStringStorageFromFileAsync(name, path, kind)` |

All methods complete asynchronously (C++: blocking with timeout; Python: blocking; Unreal/Unity: `Task`/callback).\
The `kind` parameter for Unreal uses `uint8` ordinals: `0` = `List`, `1` = `Map`, `2` = `Multimap`.\
For C++ and Python, use the typed `::Tryll::StringStorageKind_Map` / `StringStorageKind.Map` constants.

## Related

- [Canned Response node](https://docs.tryllengine.com/v0.8/reference/nodes/canned-response.html)
- [Regex Guardrail node](https://docs.tryllengine.com/v0.8/reference/nodes/regex-guardrail.html)
- [Intent to Instruction node](https://docs.tryllengine.com/v0.8/reference/nodes/intent-to-instruction.html) — the keyed consumer of Map-kind storages.
- [How to use canned responses and guardrails](https://docs.tryllengine.com/v0.8/how-to/use-canned-responses-and-guardrails.html)
- [How to build an intent-driven NPC](https://docs.tryllengine.com/v0.8/how-to/build-intent-driven-npc.html)
- [Embedded String Storage](https://docs.tryllengine.com/v0.8/reference/embedded-string-storage.html) — for vector retrieval.
- [Glossary](https://docs.tryllengine.com/v0.8/reference/glossary.html#string-storage)
