# Query Rewriting and HyDE for RAG

A plain `Retrieve → Generate` pipeline searches your knowledge base with the user's raw message. That works for a single, well-formed question, but real conversations are messier: typos, sentence fragments, and pronouns that only make sense against earlier turns ("what about *that* fish?", "can *it* live with goldfish?"). Retrieval on the raw text alone will often miss.

This how-to adds a **rewriting** step: a dedicated [`Generate`](https://docs.tryllengine.com/v0.9/reference/nodes/generate.html) node folds the conversation so far into fresh search text, and `Retrieve` searches with that instead of the raw message — while the answering `Generate` node still responds to the user's *original* message, so the reply reads naturally. Because rewriting needs to resolve pronouns and fix grammar, it uses the LLM; a model-free [`Transform`](https://docs.tryllengine.com/v0.9/reference/nodes/transform.html) node is the alternative when a purely deterministic (template-based) rewrite is enough (see the note at the end of Step 3).

The graph is the same either way. What changes — and what this page is mostly about — is **what you ask the node to write**.

Prerequisites

- A working `Retrieve → Generate` RAG graph — see [Create a simple RAG assistant](https://docs.tryllengine.com/v0.9/how-to/create-simple-rag-assistant.html) if you don't have one yet.
- Read [Slots and inter-node value passing](https://docs.tryllengine.com/v0.9/concepts/slots-and-inter-node-value-passing.html) for the `input` / `{{slot.<name>}}` mental model this how-to relies on.

## Choose a register first: query, or hypothetical answer

Embedding search compares your search text against your stored chunks. It works best when the two are written in the **same register** — so the shape of your knowledge base decides what the rewrite node should emit.

| Your chunks read like…                          | Ask the node for                                               | Known as        |
| ----------------------------------------------- | -------------------------------------------------------------- | --------------- |
| questions, FAQ entries, player utterances       | a **search query** — keywords, pronouns resolved               | query rewriting |
| statements, encyclopedia prose, lore, wiki text | a **hypothetical answer** — a passage in the chunks' own voice | HyDE            |

Most knowledge bases are the second kind. If your chunks read like

> *Betta fish are native to the shallow rice paddies and slow-moving streams of Southeast Asia…*

then you are storing **answers**, and embedding a *question* to find them is a mismatch — the well-known query/document asymmetry (it is why several embedding models ship a separate "query" prefix). Writing a hypothetical answer and embedding *that* puts both sides in the same register. The technique is called **HyDE** — Hypothetical Document Embeddings.

It does not matter that the hypothetical answer is partly wrong. It is never shown to anyone; it only has to *look like* the chunk you want to find, so that it lands near it in embedding space.

On the aquarium demo knowledge base, HyDE resolved pronoun-only follow-ups more reliably than a query rewrite of the same graph, and both beat no rewriting at all. Numbers and method are in `docs/research/workflow/rewrite-node-hyde-vs-query-measurement.md`.

You do not have to guess

This is measurable without an LLM judge. Label a handful of turns with the chunk IDs a correct retrieval must return, then compare Recall@1 across prompt variants — see [Compare dialog variants](https://docs.tryllengine.com/v0.9/how-to/compare-dialog-variants.html). Do include a **no-rewrite control**: rewriting is not always an improvement, and without the control you cannot tell.

## Step 1 — add a rewriting node

A rewrite step is just a `Generate` node whose only job is to produce search text. It should:

- **Never appear on the wire** — set `send = None`.
- **Never be replayed as history** — set `history_role = None`. It's a scratch artifact for this turn only.
- See the conversation so far — this is automatic: every `Generate` node projects the full turn history by default. (This is also what makes the node drift; see [the drift note](#the-node-reads-the-conversation-and-may-answer-it).)

Pick **one** of the two prompts below, per the table above.

### Option A — rewrite into a search query

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
    sampling=SamplingOverrides(temperature=0.0, max_tokens=48),
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

### Option B — write a hypothetical answer (HyDE)

```
rewrite = GenerateParams(
    system_prompt=(
        "You generate a hypothetical reference passage for a knowledge-base "
        "lookup. You are NOT talking to the user and your output is never "
        "shown to anyone — it is embedded and used to search.\n\n"
        "The conversation history is CONTEXT ONLY: use it to work out which "
        "subject the latest message refers to. Never reply to it.\n\n"
        "Write the one or two sentences an encyclopedia would contain if it "
        "answered the latest message.\n\n"
        "Rules:\n"
        "- Plain declarative statements, the way an encyclopedia reads. No "
        "greetings, no \"you should\", no questions, no bullet points.\n"
        "- Always name the subject explicitly, even when the user only said "
        "\"it\", \"they\" or \"that one\".\n"
        "- Include concrete detail of the kind the entry would carry, even if "
        "you must guess. A plausible guess retrieves well; vagueness does not.\n"
        "- Be brief. One or two SHORT sentences, 40 words maximum.\n\n"
        "### Example\n"
        "Earlier turns were about: zebra danio\n"
        "Latest message: wat temperatur they need\n"
        "Passage: Zebra danios are hardy fish that do well in water from "
        "64 to 77 F (18 to 25 C).\n\n"
        "Now write the passage for the latest message. Output the passage "
        "and nothing else."
    ),
    send=SendAnswer.None_,
    history_role=HistoryRole.None_,
    sampling=SamplingOverrides(temperature=0.0, max_tokens=96),
    default_exit="retrieve",
)
```

```
GenerateParamsT rewrite;
rewrite.system_prompt =
    "You generate a hypothetical reference passage for a knowledge-base "
    "lookup. Your output is never shown to anyone - it is embedded and "
    "used to search. The conversation history is CONTEXT ONLY: use it to "
    "work out which subject the latest message refers to; never reply to "
    "it. Write the one or two SHORT sentences (40 words maximum) an "
    "encyclopedia would contain if it answered the latest message. Always "
    "name the subject explicitly. Output the passage and nothing else.";
rewrite.send = Tryll::SendAnswer_None;
rewrite.history_role = Tryll::HistoryRole_None;
rewrite.default_exit = "retrieve";
```

Keep the passage short — brevity is a retrieval setting, not a style choice

Two lines do most of the work in that prompt: the worked example, and *"Be brief. One or two SHORT sentences, 40 words maximum."* Dropping the brevity rule made a 3B model roughly a third more verbose and measurably **worse** at retrieval — long passages dilute the signal, and they can run into `max_tokens` and truncate mid-sentence. Keep `max_tokens` generous enough that the passage always finishes (96 is comfortable for two short sentences), and keep the instruction telling the model not to fill it.

Use the worked example on a subject that is **not in your knowledge base**. If the example passage is a near-duplicate of a real chunk, a model that parrots the example will retrieve *the example's* chunk instead of answering the user.

## Step 2 — point Retrieve at the rewritten slot

Set `Retrieve.input` to the rewrite node's name so it searches with the rewritten text instead of `user_message`:

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

Re-check your `threshold` after switching register

`threshold` is a maximum cosine distance, and a HyDE passage is longer and differently worded than a keyword query, so the absolute distances it produces are not the ones you tuned the gate against. If you rely on `not_found_exit` to refuse out-of-scope questions, re-check that it still fires after changing the prompt.

## Step 3 — leave Generate reading the original message

Don't set `Generate.input` — leave it at its default (`user_message`), so the model answers what the user actually typed, framed with the knowledge the *rewritten* text retrieved:

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

If your rewrite is a simple **deterministic** substitution (e.g. prefixing a fixed context string, or folding in an agent variable) rather than something that needs the model, use a [`Transform`](https://docs.tryllengine.com/v0.9/reference/nodes/transform.html) node instead — it renders a Mustache template into its slot with no LLM call. `Transform` already has `send` and `history_role` fixed to `None` (it never emits or replays), so you only set its `template` and `output_name`, then point `Retrieve.input` at it exactly as above.

## The node reads the conversation, and may answer it

This is the most common surprise, so expect it rather than debugging it.

A `Generate` node projects the full turn history, and there is no parameter that turns that off — `history_role = None` controls whether *this node's output* is replayed on later turns, not whether the node *reads* the transcript. So your rewrite node is handed a chat transcript that ends in a user question, which is a very strong cue to **answer** rather than to rewrite. Small models follow the transcript's shape over the system prompt, and the effect grows as the conversation gets longer: the first couple of turns obey the prompt, and later turns start coming back as replies. They may even pick up the *formatting* of the assistant turns above them.

Two consequences worth knowing:

- **If you asked for a query, this is a bug.** A prompt that says "never answer the question" does not reliably prevent it. Add a worked example of the exact output you want (see Option A), keep `max_tokens` tight, and inspect what the node actually emitted before blaming retrieval.
- **If you asked for HyDE, this is roughly the behaviour you wanted anyway** — which is why an answer-shaped rewrite is the more robust request of the two on a statement-shaped knowledge base. It also resists a related failure: a short keyword rewrite tends to echo the *previous* turn's topic, while committing to a full sentence forces the model to commit to the current question.

Either way, **read the node's output** while tuning. With diagnostics enabled the rewrite node's text is in the turn's node trace, and the Dialog Lab inspector shows it per turn — see [Compare dialog variants](https://docs.tryllengine.com/v0.9/how-to/compare-dialog-variants.html).

## Hybrid retrieval: add a keyword line

If `Retrieve.retrieval_mode` is `Hybrid`, your search text feeds two legs: a dense (embedding) leg and a lexical BM25 leg. A HyDE passage suits the dense leg but gives BM25 a lot of prose to match on, which can pull in off-target chunks.

Ask for a keyword line after the passage, so each leg gets what it wants:

```
Format — reproduce it exactly:
Passage: <one or two short sentences an encyclopedia would contain>
Keywords: <3 to 6 comma-separated search terms, subject name first>
```

- On **Dense** retrieval the keyword line changes nothing measurable — don't bother.
- On **Hybrid** it is worth having, and a bare HyDE passage without it was the weakest variant measured. If you switch a rewrite node to HyDE, switch the keyword line on at the same time or keep the node on Dense.

Small models skip the keyword line

Stating the requirement once is not enough. A 3B model omitted the `Keywords:` line about half the time, and repeating the requirement — in the opening line, in an explicit format block, as a rule, and again in the closing instruction — only got it to roughly four turns in five. Larger models complied every time. If you need the two-part shape *guaranteed*, constrain it with a grammar instead of asking: see [Constrain output with a grammar](https://docs.tryllengine.com/v0.9/how-to/constrain-output-with-a-grammar.html), which makes a missing `Keywords:` line unsampleable rather than unlikely.

## Why this works: same-turn visibility

Even though `rewrite` never appears on the wire (`send = None`) and is never replayed on later turns (`history_role = None`), its slot is fully visible to `retrieve` and `generate` **within this turn** — same-turn visibility is unconditional (see [Slots and inter-node value passing](https://docs.tryllengine.com/v0.9/concepts/slots-and-inter-node-value-passing.html#same-turn-effects)). That's what lets you keep the rewrite invisible to the user while still using it internally.

## See also

- [Create a simple RAG assistant](https://docs.tryllengine.com/v0.9/how-to/create-simple-rag-assistant.html)
- [Retrieval-Augmented Generation (RAG)](https://docs.tryllengine.com/v0.9/concepts/rag.html) — cosine distance, `top_k`, and the `threshold`
- [Transform node](https://docs.tryllengine.com/v0.9/reference/nodes/transform.html)
- [Retrieve node](https://docs.tryllengine.com/v0.9/reference/nodes/retrieve.html)
- [Constrain output with a grammar](https://docs.tryllengine.com/v0.9/how-to/constrain-output-with-a-grammar.html)
- [Compare dialog variants](https://docs.tryllengine.com/v0.9/how-to/compare-dialog-variants.html) — measure a prompt change instead of guessing at it
- [Slots and inter-node value passing](https://docs.tryllengine.com/v0.9/concepts/slots-and-inter-node-value-passing.html)
