# Use Mustache Templates for Prompt Projection

Control exactly how retrieved knowledge and instructions land in the prompt by writing a [Mustache](https://mustache.github.io/) template on your `Generate` node's `template` parameter and choosing a `placement`.

Prerequisites

- A working agent — see [Build a chat agent with a graph](https://docs.tryllengine.com/how-to/build-chat-agent-with-graph.html).
- For RAG templates: a RAG agent that retrieves chunks — see [Create a Simple RAG Assistant](https://docs.tryllengine.com/how-to/create-simple-rag-assistant.html).

______________________________________________________________________

## How projection works

Before each `Generate` call the server builds a **Mustache context** from the current turn's data and renders the node's `template` string. The rendered text is then spliced into the prompt according to `placement`.

The context contains:

| Variable             | Type                                                      | Contents                                                                                                                                                                                                                                                                      |
| -------------------- | --------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `user_message`       | string                                                    | The raw user-message text for this turn.                                                                                                                                                                                                                                      |
| `slot.<name>`        | string                                                    | Direct lookup for any [slot](https://docs.tryllengine.com/concepts/slots-and-inter-node-value-passing.html) written so far this turn, by node/`output_name`, e.g. `{{slot.greeter}}`. Works for instruction slots, `Transform` output, and any other producing node's slot.   |
| `var.<name>`         | string                                                    | Direct lookup for a declared [Agent Variable](https://docs.tryllengine.com/concepts/agent-variables.html), e.g. `{{var.mood}}`. Scalars render as their value (`bool` as `true`/`false`); a `set<string>` renders sorted and comma-joined (a single string, not an iterable). |
| `instructions`       | list of `{name, text}`                                    | Every `kind = Instruction` slot on the current interaction (one per `Instruction`/`IntentToInstruction` node that ran).                                                                                                                                                       |
| `knowledge`          | list of `{name, has_chunks, chunks:[{id,text,distance}]}` | All `KnowledgeComponent`s from the current interaction (one per `Retrieve` node).                                                                                                                                                                                             |
| `knowledge_<source>` | list of `{id,text,distance}`                              | Direct lookup for one retriever's chunks by source label, e.g. `{{#knowledge_rag}}`.                                                                                                                                                                                          |

Each `knowledge` item carries a `has_chunks` **boolean** that is `true` when the retriever found at least one chunk and `false` when it returned nothing. Use it to gate the context preamble without the preamble accidentally repeating once per chunk:

```
{{#knowledge}}
{{#has_chunks}}Context from {{name}}:
{{#chunks}}- {{text}}
{{/chunks}}{{/has_chunks}}
{{^chunks}}No relevant information was found for this topic.{{/chunks}}
{{/knowledge}}
```

No HTML escaping

All variable values are inserted verbatim — apostrophes, quotation marks, and ampersands are **never** HTML-encoded. You do not need triple-brace syntax (`{{{variable}}}`).

______________________________________________________________________

## Placement values

`template` and `placement` frame the node's resolved **`input`** (the slot `input` selects, or `user_message` by default — see [Slots and inter-node value passing](https://docs.tryllengine.com/concepts/slots-and-inter-node-value-passing.html)). The rendered template never replaces the resolved input directly; it is inserted as a separate message *around* it:

| Value string            | Effect                                                          |
| ----------------------- | --------------------------------------------------------------- |
| `before_user_as_user`   | Emitted as an extra `user` turn **before** the resolved input.  |
| `before_user_as_system` | Emitted as a `system` turn before the resolved input (default). |
| `after_user_as_user`    | Emitted as an extra `user` turn **after** the resolved input.   |
| `after_user_as_system`  | Emitted as a `system` turn after the resolved input.            |

Don't put `{{user_message}}` (or `{{slot.<name>}}` for whatever `input` resolves to) in the template — the resolved input always appears as its own separate turn already; referencing it again in the template duplicates it.

If you want the rendered text to fully replace what the user sees — the old `in_place_of_user` placement — compose it with a [`Transform`](https://docs.tryllengine.com/reference/nodes/transform.html) node instead and have `Generate` consume it via `input`: see [Query rewriting for RAG](https://docs.tryllengine.com/how-to/query-rewriting-for-rag.html) for the pattern.

______________________________________________________________________

## Step 1 — Simple RAG template

The most common pattern: inject all retrieved chunks before the user message:

```
from tryll_client.graph import (
    GraphDescription, RetrieveParams, GenerateParams, Placement,
)

RAG_TEMPLATE = (
    "{{#knowledge}}"
    "{{name}}:\n"
    "{{#chunks}}- {{text}}\n{{/chunks}}"
    "\n{{/knowledge}}"
)

graph = (
    GraphDescription()
    .add_node("retrieve", RetrieveParams(
        embedded_string_storage="my_kb.json",
        source="docs",
        found_exit="generate",
        not_found_exit="generate",
    ))
    .add_node("generate", GenerateParams(
        template=RAG_TEMPLATE,
        placement=Placement.BeforeUserAsSystem,
        system_prompt="Answer using the context above.",
        # default_exit is "" (END) by default.
    ))
    .set_start_node("retrieve")
    .set_default_model_name("Llama 3.2 3B Instruct (Q4_K_M)")
)
```

```
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

constexpr std::string_view kRagTemplate =
    "{{#knowledge}}"
    "{{name}}:\n"
    "{{#chunks}}- {{text}}\n{{/chunks}}"
    "\n{{/knowledge}}";

RetrieveParamsT retrieveParams;
retrieveParams.embedded_string_storage = "my_kb.json";
retrieveParams.source                  = "docs";
retrieveParams.found_exit              = "generate";
retrieveParams.not_found_exit          = "generate";

GenerateParamsT genParams;
genParams.template_     = std::string{kRagTemplate};
genParams.placement     = Placement::BeforeUserAsSystem;
genParams.system_prompt = "Answer using the context above.";
// genParams.default_exit is "" (END) by default.

GraphDescription graph;
graph.AddRetrieve("retrieve", std::move(retrieveParams))
     .AddGenerate("generate", std::move(genParams))
     .SetStartNode("retrieve")
     .SetDefaultModelName("Llama 3.2 3B Instruct (Q4_K_M)");
```

______________________________________________________________________

## Step 2 — Direct per-source lookup

When your graph has multiple `Retrieve` nodes with distinct `source` labels you can access each one directly and format them differently:

```
{{#knowledge_rules}}
Rule: {{text}}
{{/knowledge_rules}}
{{#knowledge_lore}}
Lore: {{text}}
{{/knowledge_lore}}
```

Each `knowledge_<source>` section is a list of chunk objects, so you iterate over it with `{{#knowledge_rules}}…{{/knowledge_rules}}`.

______________________________________________________________________

## Step 3 — Instructions from InstructionNode

Use an `InstructionNode` to inject a changeable instruction string into the prompt without touching the graph:

```
from tryll_client.graph import (
    GraphDescription, InstructionParams, GenerateParams, Placement,
)

INSTRUCTION_TEMPLATE = "{{#instructions}}{{text}}\n{{/instructions}}"

graph = (
    GraphDescription()
    .add_node("persona", InstructionParams(
        instruction="You are a friendly guide.",
        default_exit="generate",
    ))
    .add_node("generate", GenerateParams(
        template=INSTRUCTION_TEMPLATE,
        placement=Placement.BeforeUserAsSystem,
        # default_exit is "" (END) by default.
    ))
    .set_start_node("persona")
    .set_default_model_name("Llama 3.2 3B Instruct (Q4_K_M)")
)

# Later, change persona without recreating the agent:
agent.change_param("persona", "instruction", "You are a sarcastic pirate.")
```

```
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

InstructionParamsT personaParams;
personaParams.instruction  = "You are a friendly guide.";
personaParams.default_exit = "generate";

GenerateParamsT genParams;
genParams.template_ = "{{#instructions}}{{text}}\n{{/instructions}}";
genParams.placement = Placement::BeforeUserAsSystem;
// genParams.default_exit is "" (END) by default.

GraphDescription graph;
graph.AddInstruction("persona",  std::move(personaParams))
     .AddGenerate("generate",    std::move(genParams))
     .SetStartNode("persona")
     .SetDefaultModelName("Llama 3.2 3B Instruct (Q4_K_M)");

// Later (clone-set-send via typed ChangeParams):
InstructionParamsT mut = personaBaseline;
mut.instruction = "You are a sarcastic pirate.";
agent.ChangeParams("persona", std::move(mut));
```

Use `{{slot.<name>}}` for a direct single-instruction lookup:

```
{{slot.persona}}
```

______________________________________________________________________

## Step 4 — Combine instructions and knowledge

Nothing prevents you from using both in one template:

```
{{#instructions}}{{text}}
{{/instructions}}
{{#knowledge}}{{name}}:
{{#chunks}}- {{text}}
{{/chunks}}
{{/knowledge}}
```

With `placement: before_user_as_system` the rendered block becomes a system turn immediately before the user message.

______________________________________________________________________

## Placement guidance

1. **`before_user_as_system`** is the default recommendation — most modern chat-tuned models treat system turns as highest-priority.
1. If the model ignores the context, try `before_user_as_user`.
1. The `after_*` variants are uncommon; they exist for models that prefer seeing the question before the context.
1. Need to fully replace what the model sees instead of framing around it? Use a [`Transform`](https://docs.tryllengine.com/reference/nodes/transform.html) node and `input` instead of `template`/`placement` — see [Query rewriting for RAG](https://docs.tryllengine.com/how-to/query-rewriting-for-rag.html).

______________________________________________________________________

## Verify it worked

Create the agent with `enable_diagnostics=True` (Python) / `enableDiagnostics=true` (C++). The `TurnComplete.debug_info` JSON includes the full rendered prompt for each `Generate` node, so you can inspect exactly what the model received.

______________________________________________________________________

## Common pitfalls

- **`{{user_message}}` (or the resolved-input slot) inside the template.** It always expands to the empty string when it duplicates the input the placement is already framing — the resolved input is emitted as its own separate turn regardless. Remove it from the template.
- **Unknown sections.** Mustache silently ignores unknown section names. If `{{#knowledge_rag}}` renders nothing, check that your `Retrieve` node has `source: "rag"`.
- **Empty knowledge.** When the template renders to an empty string (e.g. a `{{#has_chunks}}` block with no chunks), the server skips injecting the extra message entirely. The resolved input still goes through on its own, so the model is never left with an empty turn.

______________________________________________________________________

## Related

- [Reference: Generate node](https://docs.tryllengine.com/reference/nodes/generate.html)
- [Reference: Instruction node](https://docs.tryllengine.com/reference/nodes/instruction.html)
- [Reference: Retrieve node](https://docs.tryllengine.com/reference/nodes/retrieve.html)
- [Concept: RAG](https://docs.tryllengine.com/concepts/rag.html)
- [Concept: Projection and Token Budgets](https://docs.tryllengine.com/concepts/projection-and-token-budgets.html)
- [Concept: Slots and inter-node value passing](https://docs.tryllengine.com/concepts/slots-and-inter-node-value-passing.html)
- [How-to: Create a Simple RAG Assistant](https://docs.tryllengine.com/how-to/create-simple-rag-assistant.html)
- [How-to: Query rewriting for RAG](https://docs.tryllengine.com/how-to/query-rewriting-for-rag.html)
- [How-to: Change Agent Parameters at Runtime](https://docs.tryllengine.com/how-to/change-agent-parameters.html)
