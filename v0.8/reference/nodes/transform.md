# Transform Node

**`NodeType = Transform` (`NodeParams` union discriminator 13)**

A model-free input-composition node. It renders its Mustache `template` against the current turn's slots — `user_message`, any `slot.<name>` written earlier this turn, `knowledge_<source>`, and the `#instructions` iterable block — and writes the result into its own slot. It runs no inference and never emits anything on the wire: `send = None`, `history_role = none`, fixed.

Use it to compose or rewrite text before a downstream node consumes it via `input` — the flagship case is query rewriting: `Transform` rewrites the raw user message (fixing grammar, resolving "this"/"that" references against prior turns) into a self-contained query, which a following `Retrieve` or `Generate` node then reads via `input = <transform node name>`.

See [Slots and inter-node value passing](https://docs.tryllengine.com/v0.8/concepts/slots-and-inter-node-value-passing.html) for the full mental model, and [How-to: query rewriting for RAG](https://docs.tryllengine.com/v0.8/how-to/query-rewriting-for-rag.html) for the end-to-end recipe.

______________________________________________________________________

## Parameters

| Param         | Type            | Default | Range | Structural | Description                                                                                                                                                                                          |
| ------------- | --------------- | ------- | ----- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `template`    | str (multiline) | empty   | —     | —          | Mustache template rendered at execution time against the user_message tag, slot. tags, knowledge\_ tags, and the #instructions iterable block.                                                       |
| `output_name` | str             | empty   | —     | ✓          | Name this node's output slot is stored under. Empty = the node's own name. Structural: immutable after creation — renaming would re-wire the slot dataflow that is validated once at agent creation. |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit      | Param field    | Description                                              |
| --------- | -------------- | -------------------------------------------------------- |
| `default` | `default_exit` | Default exit target after rendering. Empty string = END. |

______________________________________________________________________

## Exit routes

| Exit      | Condition                                                  |
| --------- | ---------------------------------------------------------- |
| `default` | Taken after the template is rendered into the node's slot. |

______________________________________________________________________

## Diagnostics

When `enable_diagnostics = true`, this node contributes to `debug_info.nodes[].diagnostics`. See [Turn Diagnostics JSON](https://docs.tryllengine.com/v0.8/reference/turn-diagnostics-json.html) for the envelope.

Transform runs no model, so it emits a `parameters` section only.

| Key                      | Meaning                                                                                                                                                                     | Absent when                  |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| `parameters.template`    | The Mustache template, in full — **not** truncated the way [Generate](https://docs.tryllengine.com/v0.8/reference/nodes/generate.html#diagnostics)'s knowledge template is. | —                            |
| `parameters.output_name` | The configured output slot name.                                                                                                                                            | Defaulting to the node name. |

The node records the template but **not the rendered result**. To see what it produced, read that slot from the turn's `interaction` snapshot, or from the `input.prompt[]` of the node that consumed it.

______________________________________________________________________

## Example

```
from tryll_client.graph import GraphDescription, TransformParams, RetrieveParams, GenerateParams

graph = (
    GraphDescription()
    .add_node("rewrite", TransformParams(
        template="Rewrite the user's message as a standalone question, "
                 "resolving pronouns against earlier turns: {{user_message}}",
        default_exit="retrieve",
    ))
    .add_node("retrieve", RetrieveParams(
        input="rewrite",
        embedded_string_storage="aquarium_kb",
        found_exit="generate",
        not_found_exit="generate",
    ))
    .add_node("generate", GenerateParams(input="rewrite", default_exit=""))
    .set_start_node("rewrite")
)
```

```
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

TransformParamsT tp;
tp.template_ = "Rewrite the user's message as a standalone question: {{user_message}}";
tp.default_exit = "retrieve";

GraphDescription graph;
graph.AddTransform("rewrite", std::move(tp))
     .SetStartNode("rewrite");
```

______________________________________________________________________

## Related

- [Concept: Slots and inter-node value passing](https://docs.tryllengine.com/v0.8/concepts/slots-and-inter-node-value-passing.html)
- [How-to: Query rewriting for RAG](https://docs.tryllengine.com/v0.8/how-to/query-rewriting-for-rag.html)
- [Retrieve node](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve.html), [Generate node](https://docs.tryllengine.com/v0.8/reference/nodes/generate.html)
