# Query Rewriting for RAG

A plain `Retrieve → Generate` pipeline searches your knowledge base with the user's raw message. That works for a single, well-formed question, but real conversations are messier: typos, sentence fragments, and pronouns that only make sense against earlier turns ("what about *that* fish?", "can *it* live with goldfish?"). Retrieval on the raw text alone will often miss.

This how-to adds a **query-rewriting** step: a dedicated [`Generate`](https://docs.tryllengine.com/reference/nodes/generate.html) node folds the conversation so far into a standalone search query, and `Retrieve` searches with that instead of the raw message — while the answering `Generate` node still responds to the user's *original* message, so the reply reads naturally. Because rewriting needs to resolve pronouns and fix grammar, it uses the LLM; a model-free [`Transform`](https://docs.tryllengine.com/reference/nodes/transform.html) node is the alternative when a purely deterministic (template-based) rewrite is enough (see the note at the end of Step 3).

Prerequisites

- A working `Retrieve → Generate` RAG graph — see [Create a simple RAG assistant](https://docs.tryllengine.com/how-to/create-simple-rag-assistant.html) if you don't have one yet.
- Read [Slots and inter-node value passing](https://docs.tryllengine.com/concepts/slots-and-inter-node-value-passing.html) for the `input` / `{{slot.<name>}}` mental model this how-to relies on.

## Step 1 — add a rewriting node

A rewrite step is just a `Generate` node whose only job is to produce a standalone query. It should:

- **Never appear on the wire** — set `send = None`.
- **Never be replayed as history** — set `history_role = None`. It's a scratch artifact for this turn only.
- See the conversation so far — this is automatic: every `Generate` node projects the full turn history by default.

```
from tryll_client.graph import GraphDescription
from tryll_client._generated.node_params import GenerateParams, SendAnswer, HistoryRole, SamplingOverrides

rewrite = GenerateParams(
    system_prompt=(
        "You rewrite the user's latest message into a single, well-formed, "
        "standalone search query. Fix spelling and grammar. Resolve pronouns "
        "and vague references using the conversation so far. Output ONLY the "
        "rewritten query — no preamble, no quotes."
    ),
    send=SendAnswer.None_,
    history_role=HistoryRole.None_,
    sampling=SamplingOverrides(temperature=0.0),
    default_exit="retrieve",
)
```

```
using namespace Tryll::NodeParams;

GenerateParamsT rewrite;
rewrite.system_prompt =
    "You rewrite the user's latest message into a single, well-formed, "
    "standalone search query. Fix spelling and grammar. Resolve pronouns "
    "and vague references using the conversation so far. Output ONLY the "
    "rewritten query — no preamble, no quotes.";
rewrite.send = Tryll::SendAnswer_None;
rewrite.history_role = Tryll::HistoryRole_None;
rewrite.default_exit = "retrieve";
```

## Step 2 — point Retrieve at the rewritten slot

Set `Retrieve.input` to the rewrite node's name so it searches with the rewritten query instead of `user_message`:

```
from tryll_client._generated.node_params import RetrieveParams

retrieve = RetrieveParams(
    embedded_string_storage="my-kb.json",
    input="rewrite",          # <- read the Transform/Generate node's slot
    top_k=3,
    threshold=0.5,
    found_exit="generate",
    not_found_exit="canned_response",
)
```

```
RetrieveParamsT retrieve;
retrieve.embedded_string_storage = "my-kb.json";
retrieve.input = "rewrite";
retrieve.top_k = 3;
retrieve.threshold = 0.5f;
retrieve.found_exit = "generate";
retrieve.not_found_exit = "canned_response";
```

## Step 3 — leave Generate reading the original message

Don't set `Generate.input` — leave it at its default (`user_message`), so the model answers what the user actually typed, framed with the knowledge the *rewritten* query retrieved:

```
from tryll_client._generated.node_params import Placement

generate = GenerateParams(
    system_prompt="You are a helpful aquarium assistant.",
    template=(
        "Use the following information to answer the next question.\n\n"
        "{{#knowledge}}{{#chunks}}- {{text}}\n{{/chunks}}{{/knowledge}}"
    ),
    placement=Placement.BeforeUserAsSystem,
    send=SendAnswer.Streamed,
)
```

Wire the graph: `rewrite → retrieve → generate` (with `retrieve`'s `not_found_exit` falling back to a canned response, as usual).

Transform instead of Generate

If your rewrite is a simple **deterministic** substitution (e.g. prefixing a fixed context string, or folding in an agent variable) rather than something that needs the model, use a [`Transform`](https://docs.tryllengine.com/reference/nodes/transform.html) node instead — it renders a Mustache template into its slot with no LLM call. `Transform` already has `send` and `history_role` fixed to `None` (it never emits or replays), so you only set its `template` and `output_name`, then point `Retrieve.input` at it exactly as above.

## Why this works: same-turn visibility

Even though `rewrite` never appears on the wire (`send = None`) and is never replayed on later turns (`history_role = None`), its slot is fully visible to `retrieve` and `generate` **within this turn** — same-turn visibility is unconditional (see [Slots and inter-node value passing](https://docs.tryllengine.com/concepts/slots-and-inter-node-value-passing.html#same-turn-effects)). That's what lets you keep the rewrite invisible to the user while still using it internally.

## See also

- [Transform node](https://docs.tryllengine.com/reference/nodes/transform.html)
- [Retrieve node](https://docs.tryllengine.com/reference/nodes/retrieve.html)
- [Slots and inter-node value passing](https://docs.tryllengine.com/concepts/slots-and-inter-node-value-passing.html)
