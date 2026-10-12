# Instruction Node

**`NodeType = Instruction` (`NodeParams` union discriminator 6)**

Attaches a named instruction string to the current interaction as a [slot](https://docs.tryllengine.com/v0.8/concepts/slots-and-inter-node-value-passing.html). A downstream `Generate` node can reference it inside a [Mustache template](https://docs.tryllengine.com/v0.8/how-to/use-mustache-templates.html) via the `{{#instructions}}` list or the `{{slot.<name>}}` direct lookup.

The node itself does **not** call a model and does **not** modify the conversation history that the model sees. It only adds a `kind=Instruction` `SlotComponent` to the current turn's interaction data, which the projection stage picks up when rendering the Generate node's template.

______________________________________________________________________

## Parameters

| Param         | Type            | Default | Range | Structural | Description                                                                                                                                                                                                                                     |
| ------------- | --------------- | ------- | ----- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `instruction` | str (multiline) | empty   | —     | —          | The instruction text. Mutable.                                                                                                                                                                                                                  |
| `output_name` | str             | empty   | —     | ✓          | Name this node's output slot is stored under. Empty = the node's own name. Downstream templates look it up via slot.. Structural: immutable after creation — renaming would re-wire the slot dataflow that is validated once at agent creation. |

## Exits

Each exit is a structural string field on the node's params; its value names the target node (empty = END).

| Exit      | Param field    | Description                                                              |
| --------- | -------------- | ------------------------------------------------------------------------ |
| `default` | `default_exit` | Default exit target after attaching the instruction. Empty string = END. |

______________________________________________________________________

## Exit routes

| Exit      | Condition                                       |
| --------- | ----------------------------------------------- |
| `default` | Always taken after the instruction is attached. |

______________________________________________________________________

## Mustache exposure

Given a node named `greeter` with `instruction = "Welcome the user."`:

| Template expression                                    | Renders to                   |
| ------------------------------------------------------ | ---------------------------- |
| `{{slot.greeter}}`                                     | `Welcome the user.`          |
| `{{#instructions}}{{name}}: {{text}}{{/instructions}}` | `greeter: Welcome the user.` |

Multiple `Instruction` nodes in one graph each contribute one entry to the `{{#instructions}}` list, in the order they executed.

______________________________________________________________________

## Diagnostics

When `enable_diagnostics = true`, this node contributes to `debug_info.nodes[].diagnostics`. See [Turn Diagnostics JSON](https://docs.tryllengine.com/v0.8/reference/turn-diagnostics-json.html) for the envelope.

Instruction records a single key, and no `input`, `output` or `engine` section.

| Key                      | Meaning                                         | Absent when |
| ------------------------ | ----------------------------------------------- | ----------- |
| `parameters.instruction` | The static instruction text this node injected. | —           |

Whether that instruction reached the model is a separate question, answered by the downstream Generate node's `input.prompt[]` — an `Instruction` node that ran is not proof its text was rendered, since that depends on the consuming node's `template`.

______________________________________________________________________

## Example

```
from copy import deepcopy
from tryll_client.graph import GraphDescription, InstructionParams, GenerateParams, Placement

greeter_baseline = InstructionParams(
    instruction="You are a friendly greeter. Welcome the user warmly.",
    default_exit="generate",
)

graph = (
    GraphDescription()
    .add_node("greeter", greeter_baseline)
    .add_node("generate", GenerateParams(
        template="{{slot.greeter}}",
        placement=Placement.BeforeUserAsSystem,
        default_exit="",   # empty = END
    ))
    .set_start_node("greeter")
    .set_default_model_name("Llama 3.2 3B Instruct (Q4_K_M)")
)
agent = client.create_agent(graph)

# Change the instruction without recreating the agent:
p = deepcopy(greeter_baseline)
p.instruction = "You are a stern librarian."
agent.change_params("greeter", p)
```

```
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

InstructionParamsT ip;
ip.instruction  = "You are a friendly greeter. Welcome the user warmly.";
ip.default_exit = "generate";
InstructionParamsT greeterBaseline = ip; // keep a copy for mutations

GenerateParamsT gp;
gp.template_  = "{{slot.greeter}}";
gp.placement  = ::Tryll::Placement_BeforeUserAsSystem;
// gp.default_exit = ""; // empty = END (the default)

GraphDescription graph;
graph.AddInstruction("greeter",   std::move(ip))
     .AddGenerate("generate",     std::move(gp))
     .SetStartNode("greeter")
     .SetDefaultModelName("Llama 3.2 3B Instruct (Q4_K_M)");
auto agent = client.CreateAgent(graph);

// Change the instruction without recreating the agent:
InstructionParamsT mutated = greeterBaseline;
mutated.instruction = "You are a stern librarian.";
agent.ChangeParams("greeter", std::move(mutated));
```

______________________________________________________________________

## Related

- [How-to: Use Mustache Templates](https://docs.tryllengine.com/v0.8/how-to/use-mustache-templates.html)
- [How-to: Change Agent Parameters at Runtime](https://docs.tryllengine.com/v0.8/how-to/change-agent-parameters.html)
- [Reference: Generate node](https://docs.tryllengine.com/v0.8/reference/nodes/generate.html)
- [Concept: Projection and Token Budgets](https://docs.tryllengine.com/v0.8/concepts/projection-and-token-budgets.html)
