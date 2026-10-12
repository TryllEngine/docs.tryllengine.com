# Embedded String Storage

An **embedded string storage** is a session-owned, named, vector-indexed container of text chunks. It is the backing store for every [Retrieve node](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve.html) in a [graph](https://docs.tryllengine.com/v0.8/reference/glossary.html#graph) — the building block of [RAG](https://docs.tryllengine.com/v0.8/reference/glossary.html#rag) in Tryll.

Under the hood, each storage holds:

1. A list of **records** — `{ id, text }` pairs, optionally with metadata.
1. A precomputed [embedding](https://docs.tryllengine.com/v0.8/reference/glossary.html#embedding) per record.
1. An [HNSW](https://docs.tryllengine.com/v0.8/reference/glossary.html#hnsw) index over those embeddings, tuned for cosine distance.
1. An in-process **Okapi BM25** index over record text (persisted as a `.bm25` sidecar on Path A), used by [Retrieve](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve.html) in `Lexical` / `Hybrid` modes. Tokenization is Unicode-aware (casefold + letter/number split via utf8proc); sidecar **format version 2** — older ASCII-casefold sidecars are rebuilt on next load.

Dense retrieval uses embedding similarity. Lexical / Hybrid modes add BM25 (and RRF fusion for Hybrid). For plain substring / regex / exact-match over non-embedded lists, see [string storage](https://docs.tryllengine.com/v0.8/reference/string-storage.html).

## Node param: relative file path (preferred)

Set the node's `embedded_string_storage` parameter to a **relative path** under the process storage root:

```
embedded_string_storage = "rag/adventure_quests.json"
```

The path is resolved at `CreateAgentRequest` time against the process storage root, set at server startup (see [Server Configuration → Storage root](https://docs.tryllengine.com/v0.8/reference/server-configuration.html#storage-root)). There is no per-session root; a path that resolves outside it is rejected with `3009 StorageOutsideRoot`. The server loads the KB config file, builds or reuses a cached HNSW index, and Pins the entry so it survives eviction for the lifetime of the agent.

## Creation paths (explicit Pin)

`CreateEmbeddedStringStorageRequest` supports two construction paths. Exactly one is used per request.

| Path                | Trigger             | What the server does                                                                                                                                                                                                                                |
| ------------------- | ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **A** (file-backed) | `config_path` set   | Relative path resolved against the process storage root. Loads the KB config JSON, resolves `records_file` and optional `index_file` / `bm25_file` relative to the config's directory, builds or loads the HNSW + BM25 indexes, and Pins the entry. |
| **B** (inline)      | `strings` non-empty | Embeds every string with `embedding_model` and builds an in-memory HNSW index. Registered as a virtual entry under `name`.                                                                                                                          |

`embedding_model` is **required** for Path B (inline strings). For Path A (file-backed), the model is taken from the KB config file; the wire `embedding_model` field is ignored.

### Path A — `config_path`

The config is a UTF-8 JSON file with this shape:

```
{
    "version": 1,
    "embedding_model": "All-MiniLM-L6-v2 (Q4_K_M)",
    "records_file": "adventure_quests.kb.json",
    "index_file": "adventure_quests.kb.usearch",
    "bm25_file": "adventure_quests.kb.bm25",
    "bm25_tokenizer": "basic",
    "fields": [
        { "name": "min_level",         "type": "int",         "default": 0      },
        { "name": "character_class",   "type": "string",      "optional": true  },
        { "name": "character_classes", "type": "set<string>", "optional": true  }
    ]
}
```

| Field             | Type   | Required | Description                                                                                                                                                                                                                                                                                                                                                               |
| ----------------- | ------ | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `version`         | int    | No       | Schema version. Currently `1`.                                                                                                                                                                                                                                                                                                                                            |
| `embedding_model` | string | Yes      | Catalog name of the embedding model. Must exist in `models.json` with `"model_type": "embedding"`.                                                                                                                                                                                                                                                                        |
| `records_file`    | string | Yes      | Relative path (from the config file's directory) to the records JSON.                                                                                                                                                                                                                                                                                                     |
| `index_file`      | string | No       | Relative path to a pre-built `*.usearch` HNSW index. If absent, the server builds the index at load time from the records and their embeddings.                                                                                                                                                                                                                           |
| `bm25_file`       | string | No       | Relative path to a persisted BM25 sidecar. If omitted and `index_file` is set, defaults to the same stem with a `.bm25` extension (e.g. `adventure_quests.kb.usearch` → `adventure_quests.kb.bm25`). First load **builds and writes** the sidecar; later loads **reuse** it until the records file is newer, or format / tokenizer / doc-count mismatch forces a rebuild. |
| `bm25_tokenizer`  | string | No       | `"basic"` (default) or `"enhanced"`. Both modes Unicode-casefold via utf8proc, then split on non-letter/non-number. `"enhanced"` additionally drops a small English stopword set and applies a trivial suffix strip. Stored in the sidecar header; changing it forces a rebuild. Format version **2** (utf8proc) rebuilds any version-1 ASCII sidecar automatically.      |
| `fields`          | array  | No       | Typed metadata schema — field declarations for filterable record metadata. When present, each record's `metadata` object is validated against it at load time. See [Metadata schema](#metadata-schema).                                                                                                                                                                   |

The **records file** (`*.kb.json` by convention) is a JSON array of objects:

```
[
    {
        "id":   "ai_storytellers_1",
        "text": "The AI storyteller is the main mechanism used to ...",
        "metadata": { "source": "…", "link": "…" }
    },
    {
        "id":   "ai_storytellers_2",
        "text": "…"
    }
]
```

Per record:

| Field      | Type   | Required | Description                                                                                                                                                                                                                            |
| ---------- | ------ | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`       | string | Yes      | Stable identifier; returned on retrieval so clients can map back to the source.                                                                                                                                                        |
| `text`     | string | Yes      | The chunk text, already segmented to a size that fits within the embedding model's context window.                                                                                                                                     |
| `metadata` | object | No       | Optional typed metadata object. When the config declares a `fields` schema, each record's metadata is validated and stored in typed slots used by the Retrieve node's `filter` predicate. Fields not listed in the schema are ignored. |

If an `index_file` is present, its dimension must match `embedding_model`'s output dimension; a mismatch is rejected with error `8003` `InvalidEmbeddedStringStorageData`.

## Metadata schema

When a config includes a `fields` array, the server validates every record's `metadata` object against it at load time and stores the values in typed per-field slots. Those slots are what the Retrieve node's [`filter` predicate](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve-filter-grammar.html) compares against during vector search — the filter compiler resolves field names to slot indices once at compile time, so no string lookups happen on the hot path.

### Field declarations

Each entry in `fields` declares one filterable field:

| Key          | Required                  | Description                                                                                                               |
| ------------ | ------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `name`       | Yes                       | Field identifier used in records and in filter references (`{"knowledge": "<name>"}`).                                    |
| `type`       | Yes                       | One of `int`, `float`, `string`, `bool`, `set<string>`.                                                                   |
| `default`    | Exclusive with `optional` | Value applied when a record omits this field. Must be coercible to `type`.                                                |
| `optional`   | Exclusive with `default`  | `true` — missing field is allowed; filters referencing it on a record where it is absent evaluate to `false`.             |
| `filterable` | No (default `true`)       | If `false`, the field is loaded but cannot be referenced in filters. Useful for display-only fields such as `page_title`. |

Exactly one of `default` or `optional: true` must be present — a field that is neither is a load-time error.

### Supported types

| Type          | JSON value in records | Notes                                                                  |
| ------------- | --------------------- | ---------------------------------------------------------------------- |
| `int`         | number (integer)      | Stored as `int64`.                                                     |
| `float`       | number                | Stored as `double`. `int`↔`float` promotion is allowed in comparisons. |
| `string`      | string                |                                                                        |
| `bool`        | `true` / `false`      |                                                                        |
| `set<string>` | array of strings      | Stored sorted and deduplicated. Used as the `haystack` of an `in` op.  |

### Missing-field semantics

When a field is declared `optional: true` and a record omits it, every comparison or `in` operation that references that field on that record evaluates to **false**. The field is not treated as "no restriction" — filter authors who want an unrestricted fallback should encode it explicitly with `or` and a separate boolean gating field.

### Example

```
{
    "version": 1,
    "embedding_model": "All-MiniLM-L6-v2 (Q4_K_M)",
    "records_file": "adventure_quests.kb.json",
    "index_file":   "adventure_quests.kb.usearch",
    "fields": [
        { "name": "topic",             "type": "string",      "default": ""       },
        { "name": "min_level",         "type": "int",         "default": 0        },
        { "name": "tier",              "type": "string",      "default": "common" },
        { "name": "character_class",   "type": "string",      "optional": true    },
        { "name": "character_classes", "type": "set<string>", "optional": true    },
        { "name": "quest_reached",     "type": "string",      "optional": true    }
    ]
}
```

A record that uses this schema might look like:

```
{
    "id":   "warrior_combat_intro",
    "text": "The sword-and-shield style favoured by Aldhaven warriors …",
    "metadata": {
        "topic":             "Combat",
        "min_level":         1,
        "character_classes": ["warrior", "paladin"]
    }
}
```

`topic`, `min_level`, and `tier` use defaults when absent. `character_class`, `character_classes`, and `quest_reached` are optional and will evaluate to `false` in any filter that references them on a record that omits them.

See [Retrieve filter grammar](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve-filter-grammar.html) for the full JSON predicate syntax that queries these fields.

### Path B — inline `strings`

For programmatic small-scale use, the client sends strings directly:

| Field             | Type       | Required | Description                                                        |
| ----------------- | ---------- | -------- | ------------------------------------------------------------------ |
| `name`            | string     | Yes      | Unique name within the session.                                    |
| `strings`         | `[string]` | Yes      | The chunks. The server assigns sequential `id`s (`"0"`, `"1"`, …). |
| `embedding_model` | string     | Yes      | Catalog name of the embedding model.                               |

The server embeds every string at creation time and builds the index in memory. Nothing is written to disk; the storage is gone once the session ends.

## Lifecycle

**The preferred approach is a relative file path on the node param** — no explicit `Create` / `Destroy` is needed. The server caches the loaded index and Pins it when the agent is created; the cache entry reverts to OnDemand (eligible for eviction) when the agent is destroyed.

Use explicit `CreateEmbeddedStringStorageRequest` / `DestroyEmbeddedStringStorageRequest` only when you need to share the same index across multiple agents or supply inline strings (Path B).

| Step               | Frame                                                           | Notes                                                                                                                                                                                                                   |
| ------------------ | --------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Lazy load**      | *(none)*                                                        | Set `embedded_string_storage = "relative/kb.json"` on the node param. The server resolves, loads, and Pins the index at `CreateAgentRequest` time.                                                                      |
| **Explicit Pin**   | `CreateEmbeddedStringStorageRequest`                            | Pins the storage so it persists beyond a single agent creation. Server responds with `CreateEmbeddedStringStorageResponse` carrying `record_count` and `embedding_dim`.                                                 |
| **Use**            | *(referenced by Retrieve node `embedded_string_storage` param)* | Scoped to the [session](https://docs.tryllengine.com/v0.8/reference/glossary.html#session). For a Path A storage the param value is the relative `config_path`; for a Path B storage it is the `name` used at creation. |
| **Explicit Unpin** | `DestroyEmbeddedStringStorageRequest`                           | For file-backed (Path A) entries: demotes from Pinned to OnDemand, eligible for eviction once no agent holds it. For virtual (Path B) entries: removes the named entry. Server responds with `Ack`.                     |

See [Lifetime and Ownership → EmbeddedStringStorage](https://docs.tryllengine.com/v0.8/concepts/lifetime-and-ownership.html#embeddedstringstorage) for the full ownership model and the separation between the on-disk index cache and the in-memory index object.

## Consumption by Retrieve

A [Retrieve node](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve.html) names the storage via its `embedded_string_storage` param. Per turn, the node:

1. Embeds the user's message with the embedding model recorded in the KB config file referenced by `embedded_string_storage`.
1. Queries the storage's HNSW index for the `top_k` closest records by cosine distance. If a `filter` is set on the node, each candidate record's metadata is evaluated against the compiled predicate during the search; records that do not match are excluded before `top_k` is applied.
1. Drops results whose distance exceeds `threshold`.
1. Emits one [knowledge component](https://docs.tryllengine.com/v0.8/reference/glossary.html#knowledge-component) per surviving record onto the current [interaction](https://docs.tryllengine.com/v0.8/reference/glossary.html#interaction).

How those components appear in the [prompt](https://docs.tryllengine.com/v0.8/reference/glossary.html#prompt) is controlled by the downstream `Generate` node's `template` (Mustache) and `placement` params — see [Use Mustache Templates](https://docs.tryllengine.com/v0.8/how-to/use-mustache-templates.html).

## Validation and errors

The server validates at creation time:

- Exactly one of `config_path` / `strings` must be set.
- Path B: the wire `embedding_model` must be in the catalog with `"model_type": "embedding"`.
- Path A: the KB config's `embedding_model` must be in the catalog with `"model_type": "embedding"`.
- Path A: `records_file` must exist, parse, and be non-empty.
- Path A with `index_file`: the index must load and its dimension must match the embedding model.
- Path B: `strings` must be non-empty.

| Code                                                                                                | Cause                                                                                                                            |
| --------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| [`8001`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#8xxx-embedded-string-storage) | Name empty, malformed, or reserved.                                                                                              |
| [`8002`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#8xxx-embedded-string-storage) | A storage with this name already exists in the session.                                                                          |
| [`8003`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#8xxx-embedded-string-storage) | Content rejected: missing / corrupt config or records, empty inline array, unknown embedding model, or index-dimension mismatch. |

## Minimum working example

```
# Path B (inline)
client.create_embedded_string_storage(
    name="aquarium_kb",
    strings=[
        "Neon tetras thrive in soft, slightly acidic water.",
        "Goldfish are cold-water fish and should not share a tank with tropicals.",
    ],
    embedding_model="All-MiniLM-L6-v2 (Q4_K_M)",
)

# Path A (file-backed) — relative path resolved against the process storage root
client.create_embedded_string_storage(
    name="rimworld_kb",
    config_path="rag/rimworld.json",
    embedding_model="All-MiniLM-L6-v2 (Q4_K_M)",
)
```

```
// Path B (inline)
client.CreateEmbeddedStringStorageFromStrings(
    "aquarium_kb",
    {
        "Neon tetras thrive in soft, slightly acidic water.",
        "Goldfish are cold-water fish and should not share a tank with tropicals.",
    },
    "All-MiniLM-L6-v2 (Q4_K_M)");

// Path A (file-backed) — relative path resolved against the process storage root
auto info = client.CreateEmbeddedStringStorage(
    "rimworld_kb",
    "rag/rimworld.json",
    /*embeddingModel=*/"All-MiniLM-L6-v2 (Q4_K_M)");
```

```
auto* subsystem = GetGameInstance()->GetSubsystem<UTryllSubsystem>();

// Path B (inline) — requires EmbeddingModel.
subsystem->RequestCreateEmbeddedStringStorageFromStrings(
    TEXT("aquarium_kb"),
    { TEXT("Neon tetras thrive in soft, slightly acidic water."),
      TEXT("Goldfish are cold-water fish and should not share a tank with tropicals.") },
    TEXT("All-MiniLM-L6-v2 (Q4_K_M)"));

// Path A (file-backed). Relative path; model comes from the KB config.
subsystem->RequestCreateEmbeddedStringStorage(
    TEXT("rimworld_kb"),
    TEXT("rag/rimworld.json"));

// Bind OnCreateEmbeddedStringStorageComplete(Name, RecordCount, bSuccess)
// to continue once the index is built.
```

See the full workflow in [How to create a simple RAG assistant](https://docs.tryllengine.com/v0.8/how-to/create-simple-rag-assistant.html).

## Client bindings

- **C++:** `Tryll::Client::TryllClient::CreateEmbeddedStringStorage` (file config) / `CreateEmbeddedStringStorageFromStrings` (inline) — `TryllClient.h`
- **Python:** `tryll_client.TryllClient.create_embedded_string_storage` (pass either `config_path=…` or `strings=…`) — `client.py`
- **Unreal:** `UTryllSubsystem::RequestCreateEmbeddedStringStorage` — `TryllSubsystem.h`

## Related

- [Retrieve node](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve.html)
- [Retrieve filter grammar](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve-filter-grammar.html)
- [Concept: RAG](https://docs.tryllengine.com/v0.8/concepts/rag.html)
- [How to create a simple RAG assistant](https://docs.tryllengine.com/v0.8/how-to/create-simple-rag-assistant.html)
- [String Storage](https://docs.tryllengine.com/v0.8/reference/string-storage.html) — non-embedded variant.
- [Glossary](https://docs.tryllengine.com/v0.8/reference/glossary.html#embedded-string-storage)
