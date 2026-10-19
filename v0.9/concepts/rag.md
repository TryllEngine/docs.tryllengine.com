# Retrieval-Augmented Generation (RAG)

Small local models know general English. They do not know your game's lore, your product's changelog, or the contents of a customer's ticket history. RAG is the standard trick for fixing that: before the model answers, find the handful of most-relevant chunks of your data, and paste them into the prompt. This page explains how Tryll's RAG works end to end and when to use it.

## The moving parts

```
flowchart LR
    subgraph Session["Session"]
        ESS["Embedded string storage<br>(per-session, named)"]
    end
    subgraph Agent["Agent turn"]
        U["User message"] --> R["Retrieve node<br>embed + search"]
        R -- "chunks" --> KC["Knowledge on the turn"]
        KC --> P["Projection<br>weaves chunks into prompt"]
        P --> G["Generate node"]
    end
    R -. "reads" .-> ESS
```

Tryll splits RAG into two phases:

1. **Index once.** Your data is embedded and stored in an [embedded string storage](https://docs.tryllengine.com/v0.9/reference/embedded-string-storage.html) — a named, per-session HNSW index over the embeddings.
1. **Retrieve every turn.** The [`Retrieve` node](https://docs.tryllengine.com/v0.9/reference/nodes/retrieve.html) searches with the current user message (by embedding, keywords, or both), finds the top-K best-matching chunks, and attaches them to the current turn. The downstream `Generate` node's projection picks them up and rewrites the user turn to include them.

Nothing else changes. There is no special "RAG model", no separate server. A RAG agent is an agent with a `Retrieve` node in front of a `Generate` node.

## What an embedded string storage actually stores

Two things:

- a **records file** (`*.kb.json`) — an array of `{ id, text, metadata? }` entries, each a chunk you want retrievable;
- optionally an **on-disk index file** (`*.usearch`) — the HNSW graph for fast top-K lookup.

The index is rebuilt from scratch when it is missing, older than the records file, or for a different embedding model. When it is present and fresh, storage creation is near-instant.

You can create a storage two ways:

- **Path A — config on disk.** Ship a tiny `*.json` with pointers to a records file and an optional index file. Use this when the data lives alongside the server. See [Embedded String Storage](https://docs.tryllengine.com/v0.9/reference/embedded-string-storage.html).
- **Path B — inline strings.** Pass a `strings[]` array and an `embedding_model` on the wire. Use this when the data is produced at runtime. Fine for hundreds or low thousands of chunks; for bigger corpora, go with Path A and avoid re-embedding every reconnect.

## Search modes, top-K, and the threshold

The `Retrieve` node's `retrieval_mode` picks how candidates are ranked:

| Mode              | Ranking                                                                                         |
| ----------------- | ----------------------------------------------------------------------------------------------- |
| `Dense` (default) | HNSW vector search by **cosine distance** to the query embedding.                               |
| `Lexical`         | BM25 keyword scoring only. No embedding distance.                                               |
| `Hybrid`          | Runs both, then fuses the two ranked lists with Reciprocal Rank Fusion (RRF, constant `rrf_k`). |

In dense search, for two unit vectors the distance is `1 − cosine_similarity`, so:

| Distance | Similarity             |
| -------- | ---------------------- |
| 0.0      | identical direction    |
| 0.2      | quite close            |
| 0.5      | loosely related        |
| 1.0      | orthogonal (unrelated) |
| 2.0      | opposite direction     |

The `Retrieve` node returns the `top_k` best-ranked chunks. Setting a `threshold` drops dense results with distance *above* it — i.e., the threshold is an upper bound on distance (lower bound on similarity). In `Hybrid` mode it gates only the dense candidates before fusion, and in `Lexical` mode it is ignored.

If *no* chunks survive, the node exits via `not_found` instead of `found`. Your graph decides what to do in that case — the most common pattern is to still route to `Generate` and let the template render gracefully with no chunks (empty `{{#knowledge}}` sections collapse to nothing, leaving only `system_prompt` and `user_message`).

## Metadata filtering

Beyond cosine similarity and the distance threshold, records can be **gated by typed metadata** declared in the knowledge base config's `fields` array. The Retrieve node's `filter` parameter accepts a JSON predicate that is compiled against that schema and evaluated per-candidate during vector search — records that do not match are excluded before `top_k` and `threshold` are applied.

This lets you answer questions like:

- "Only return lore the player's character class can access."
- "Only return quests the player's level is high enough to receive."
- "Only return lore entries the player has already unlocked."

The client embeds its own state as JSON literals when building the filter, and calls `change_param` to push an updated filter whenever the relevant state changes:

```
agent.change_param("retrieve", "filter", json.dumps({
    "op": "and", "args": [
        {"op": "ge",
         "lhs": {"value": player.level},
         "rhs": {"knowledge": "min_level"}},
        {"op": "in",
         "needle":   {"value": player.character_class},
         "haystack": {"knowledge": "character_classes"}},
    ]
}))
```

For the full grammar and all operator shapes, see [Retrieve filter grammar](https://docs.tryllengine.com/v0.9/reference/nodes/retrieve-filter-grammar.html). For how to declare the schema, see [Embedded String Storage — Metadata schema](https://docs.tryllengine.com/v0.9/reference/embedded-string-storage.html#metadata-schema).

## Where the chunks land in the prompt

Tryll does *not* fuse retrieved text into the system prompt and hope. You write a [Mustache](https://mustache.github.io/) template on the downstream `Generate` node's `template` param and pick a `placement`.

The projection stage builds a context from the current turn's data:

| Variable                                          | Contents                                                                                                                                                           |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `{{#knowledge}}…{{/knowledge}}`                   | Iterates over every retriever's result block. Inside: `{{name}}` (source label) and `{{#chunks}}…{{/chunks}}` (each chunk's `{{id}}`, `{{text}}`, `{{distance}}`). |
| `{{#knowledge_<source>}}…{{/knowledge_<source>}}` | Direct per-retriever iteration — e.g. `{{#knowledge_rag}}`.                                                                                                        |
| `{{user_message}}`                                | The raw user message.                                                                                                                                              |

The `placement` param on `GenerateNode` controls where the rendered text appears relative to the resolved input — it is always inserted as a separate turn *around* the input, never in place of it:

| Value                                         | Effect                                                                                 |
| --------------------------------------------- | -------------------------------------------------------------------------------------- |
| `before_user_as_system`                       | Rendered text becomes a system turn before the user message. **(recommended default)** |
| `before_user_as_user`                         | Rendered text becomes a user turn before the user message.                             |
| `after_user_as_system` / `after_user_as_user` | Appended after the user message.                                                       |

To fully replace what the model sees instead of framing it, compose the rendered text with a [`Transform`](https://docs.tryllengine.com/v0.9/reference/nodes/transform.html) node and have `Generate` consume it via `input` — see [Use Mustache Templates](https://docs.tryllengine.com/v0.9/how-to/use-mustache-templates.html#placement-values) for the pattern.

Why this is worth caring about: the single biggest lever on RAG quality is *how clearly the model can tell the difference between "here is context" and "here is the question"*. The template gives you full control over that framing, per-model, without touching any node code.

For step-by-step examples see the [Use Mustache Templates how-to](https://docs.tryllengine.com/v0.9/how-to/use-mustache-templates.html).

## Chunking is your problem (mostly)

Tryll does not chunk your source data. A record in the records file is a chunk — one thing the index can return. How you split your source text into records is up to you, and it is the part of RAG with the biggest quality impact. Rough guidelines:

- Keep chunks small enough that the top-K you intend to retrieve fits well inside the model's context window, with room for the conversation history and the answer.
- Prefer chunks with natural semantic boundaries: a paragraph, a Markdown heading section, one FAQ entry, one lore snippet. Chunks that straddle topics confuse the retriever.
- Include stable context inside the chunk itself — a section title, an entity name — so the chunk is meaningful in isolation. The retriever cannot see the surrounding document.

## When RAG helps, and when it hurts

RAG helps when:

- the answer depends on facts the base model cannot know (private data, recent events, internal documents),
- the corpus is large enough that you cannot paste it whole into the prompt, but small enough that the right few chunks will usually be among the top K,
- you want to give the user a feeling of "the assistant has read the docs".

RAG hurts when:

- the question is general reasoning and the retrieved chunks are unrelated noise — the model will often try to use them anyway and produce a worse answer than it would have without them,
- your corpus is tiny (a few hundred lines total). It is cheaper to paste the whole thing into the system prompt than to embed and retrieve.
- you expect the model to synthesise across many chunks — with top-K=3 and a small model, synthesis quality will be limited.

A sensible default for a first RAG agent: `k=3`, `threshold` in the 0.3–0.6 range (tune by watching the retriever log), `placement` = `before_user_as_system`, a template like `{{#knowledge}}{{name}}:\n{{#chunks}}- {{text}}\n{{/chunks}}{{/knowledge}}`, and a `system_prompt` that says "Use the context above to answer the user's question. If the context does not contain the answer, say so."

## Edges and pitfalls

- **Wire / config embedding-model mismatch** fails the index build with [error 8003 `EmbeddedStorageBuildFailed`](https://docs.tryllengine.com/v0.9/reference/error-codes.html#8xxx-embedded-string-storage). Match the two, or leave one of them unset. (`8002` is a different error — a storage of that name already exists.)
- **Stale indexes** are rebuilt. If you bump your records file, the next storage creation detects the mtime and re-embeds. No cache poisoning.
- **Per-session ownership.** Storages die with the session. For a stable corpus across reconnects, re-create from the same config on connect; the on-disk `.usearch` makes that cheap.
- **Knowledge does not persist across turns.** Each turn, only the most recently retrieved chunks are projected. Anything retrieved on earlier turns is not re-surfaced to the model.

## Related

- [Reference: Embedded String Storage](https://docs.tryllengine.com/v0.9/reference/embedded-string-storage.html)
- [Reference: Retrieve node](https://docs.tryllengine.com/v0.9/reference/nodes/retrieve.html)
- [Reference: Retrieve filter grammar](https://docs.tryllengine.com/v0.9/reference/nodes/retrieve-filter-grammar.html)
- [Reference: Generate node](https://docs.tryllengine.com/v0.9/reference/nodes/generate.html)
- [Concept: Models and Inference Engines](https://docs.tryllengine.com/v0.9/concepts/models-and-inference-engines.html)
- [How-to: Create a simple RAG assistant](https://docs.tryllengine.com/v0.9/how-to/create-simple-rag-assistant.html)
- [How-to: Use Mustache templates](https://docs.tryllengine.com/v0.9/how-to/use-mustache-templates.html)
- [How-to: Change Agent Parameters at Runtime](https://docs.tryllengine.com/v0.9/how-to/change-agent-parameters.html)
