# Use a LoRA Adapter

Run an LLM node with a **LoRA adapter**: a small GGUF file, fine-tuned for one base model, that changes how that model behaves for one job. A small adapter can teach a small model a narrow task, such as a fixed rewrite style or one character's way of speaking, without shipping a second model.

Any node that runs a language model takes adapters: [Generate](https://docs.tryllengine.com/v0.9/reference/nodes/generate.html), [Generate and Speak](https://docs.tryllengine.com/v0.9/reference/nodes/generate-and-speak.html), [Tool Call](https://docs.tryllengine.com/v0.9/reference/nodes/tool-call.html), [Classify Intent (LLM)](https://docs.tryllengine.com/v0.9/reference/nodes/classify-intent-llm.html), [Classify Attributes (LLM)](https://docs.tryllengine.com/v0.9/reference/nodes/classify-attributes-llm.html) and [Summarize](https://docs.tryllengine.com/v0.9/reference/nodes/summarize.html). The base model is loaded once. Each node that names an adapter runs it on its own context, so one agent can run a fine-tuned node next to nodes that use the plain base model.

Getting an adapter

Tryll does not train adapters, and this page does not cover fine-tuning. If you need an adapter for your game, [contact the Tryll developers](https://tryllengine.com/).

Prerequisites

- A LoRA adapter in **GGUF** format, trained for the **same base model** that the node runs. An adapter made for another model fails agent creation.
- That base model in the catalog and downloaded — see [Manage Models in the Editor](https://docs.tryllengine.com/v0.9/how-to/manage-models-in-editor.html).
- The server's **process storage root**. Unity and Unreal default it to `<Project>/TryllStorage/`. See [Server Configuration → Storage root](https://docs.tryllengine.com/v0.9/reference/server-configuration.html#storage-root).

## 1. Put the adapter in the storage folder

The adapter's path is **relative to the storage root**, like voice clips and string storages:

```
<storage root>/
  └─ lora/
      └─ merchant_style.gguf
```

The file stays on the machine that runs the Tryll server. The client only sends the path. For a packaged build, ship the folder with the game, as described in [Ship Storage Folders for Builds](https://docs.tryllengine.com/v0.9/how-to/ship-storage-folders-for-builds.html).

## 2. Add the adapter to the node

Each LLM node has an **Adaptation** table with a **Loras** list. Every entry has a **Path** and a **Scale**.

1. Open the workflow in the [graph editor](https://docs.tryllengine.com/v0.9/how-to/edit-workflows-in-graph-editor.html) (or select the workflow asset) and select the LLM node.
1. Expand **Adaptation → Loras** and add an element.
1. In **Path**, click the **…** button and pick the `.gguf` file. The field stores the storage-relative path, such as `lora/merchant_style.gguf`. You can also type it.
1. Leave **Scale** at `1` for now.

The workflow validator flags an absolute path or a `..` path as an error before Play, and a missing or non-`.gguf` file as a warning.

In code:

```
using Tryll.Client;

var gen = new TryllGenerateParams { SystemPrompt = "You are a merchant." };
gen.Adaptation.Loras = new[]
{
    new TryllLoraAdapter { Path = "lora/merchant_style.gguf", Scale = 1f },
};
```

1. Open the workflow asset in the [graph editor](https://docs.tryllengine.com/v0.9/how-to/edit-workflows-in-unreal-graph-editor.html) and select the LLM node.
1. In the Details panel, expand **Adaptation → Loras** and add an element.
1. In **Path**, click the browse button and pick the `.gguf` file. The stored value is the storage-relative path.
1. Leave **Scale** at `1` for now.

In C++:

```
UTryllGenerateParams* P = UTryllNodeParamsFactory::MakeGenerateParams(this);

FTryllLoraAdapter Lora;
Lora.Path.Path = TEXT("lora/merchant_style.gguf");
Lora.Scale = 1.0f;
P->Adaptation.Loras.Add(Lora);
```

```
from tryll_client.graph import GenerateParams, LoraAdapter, ModelAdaptation

gen = GenerateParams(
    system_prompt="You are a merchant.",
    adaptation=ModelAdaptation(loras=[
        LoraAdapter(path="lora/merchant_style.gguf", scale=1.0),
    ]),
)
```

```
using namespace Tryll::NodeParams;

GenerateParamsT gp;
gp.system_prompt = "You are a merchant.";

auto lora = std::make_unique<LoraAdapterT>();
lora->path  = "lora/merchant_style.gguf";
lora->scale = 1.0f;
gp.adaptation = std::make_unique<ModelAdaptationT>();
gp.adaptation->loras.push_back(std::move(lora));
```

The other LLM nodes take the same `adaptation` field.

## 3. Create the agent and check it

Create the agent as usual. The server loads the adapter when it creates the node's context, and logs each load with its size. If the path or the file is wrong, agent creation fails, and the error names the file:

| Error                         | Cause                                                                                                                                                            |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `3009 StorageOutsideRoot`     | The path is absolute or leaves the storage root with `..`.                                                                                                       |
| `3010 StorageFileNotFound`    | No file at that path under the storage root.                                                                                                                     |
| `3003 GraphCompilationFailed` | The file is not a LoRA GGUF, or it was trained for a different base model. Also: an empty path, the same file listed twice, or more than 8 adapters on one node. |

Then talk to the agent and compare its replies with the base model's. Setting **Scale** to `0` turns the adapter off without changing the graph, which gives you a quick before-and-after.

## 4. Tune the scale

**Scale** sets how strongly the adapter changes the model. The range is `0` to `4`:

- `1` applies the adapter **as trained**. Start here.
- `0` turns it **off**. The node runs the plain base model.
- Between `0` and `1` the effect is **weaker**. Lower the scale when the adapter takes over too much: replies all sound alike, or the node starts ignoring its system prompt or the character's details.
- Above `1` the effect is **stronger**. Raise it carefully, in small steps, when the adapter's effect is too faint. High values tend to make the output repetitive or broken.

Change the scale in steps of about `0.25` and compare the same dialog each time. The [Dialog Lab](https://docs.tryllengine.com/v0.9/how-to/compare-dialog-variants.html) does this well: give each variant its own agent with a different scale, run the same script with several seeds, and compare the replies side by side.

The adapter is fixed when the agent is created

`adaptation` is **structural**. Changing the path or the scale on a live agent with [Change Agent Parameters](https://docs.tryllengine.com/v0.9/how-to/change-agent-parameters.html) fails with `3006 ParamNotMutable`. To try another scale, recreate the agent.

## Several adapters on one node

A node takes **up to 8 adapters**. Each one adds its change, multiplied by its scale, to the base model. The changes add up, so the order of the list does not matter. Each file may appear only once per node.

Stacking adapters that were trained separately can weaken each one or make them conflict. If you stack adapters, retune their scales together.

## Memory

An adapter's weights are loaded next to the base model's weights, on the GPU when the model runs there. The cost is roughly the adapter's file size. A rank-16 adapter for a 3–4B model is about 50–100 MiB. Each file is loaded once and shared by every node and agent that names it, and it is freed when the last of them is gone. The [Memory window](https://docs.tryllengine.com/v0.9/how-to/inspect-server-memory.html) counts adapters with their model.

## Related

- [Reference: Generate](https://docs.tryllengine.com/v0.9/reference/nodes/generate.html) — the `adaptation` field, which every LLM node shares
- [Change Agent Parameters](https://docs.tryllengine.com/v0.9/how-to/change-agent-parameters.html) — structural fields
- [Ship Storage Folders for Builds](https://docs.tryllengine.com/v0.9/how-to/ship-storage-folders-for-builds.html)
- [Compare Dialog Variants in the Lab](https://docs.tryllengine.com/v0.9/how-to/compare-dialog-variants.html)
- [Concept: Models and Inference Engines](https://docs.tryllengine.com/v0.9/concepts/models-and-inference-engines.html)
