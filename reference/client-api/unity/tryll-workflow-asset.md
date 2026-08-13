# TryllWorkflowAsset

**Type:** `ScriptableObject`\
**Namespace:** `Tryll.Client`\
**Source:** `Runtime/TryllWorkflowAsset.cs`

`ScriptableObject` that stores a `TryllGraphDescription` as a project asset. Assign it to a [`TryllAgentComponent`](https://docs.tryllengine.com/reference/client-api/unity/tryll-agent-component.html) to define which workflow graph the agent runs. Because it is a regular Unity asset, it is serialized, version-controlled, and reusable across multiple GameObjects.

______________________________________________________________________

## Creating an asset

In the Project window: **right-click → Create → Tryll → Workflow Asset**.

The asset opens in the Inspector where you can configure `Graph` nodes and routes, or populate them programmatically via [`TryllGraphBuilder`](https://docs.tryllengine.com/reference/client-api/unity/index.html#tryllgraphbuilder).

______________________________________________________________________

## Public API

```
public sealed class TryllWorkflowAsset : ScriptableObject
{
    public TryllGraphDescription Graph;
    public List<TryllVariableDecl> Variables;
}
```

`Graph` is the complete workflow graph description. It is serialized by Unity's standard serialization and editable in the Inspector.

`Variables` declares the agent's [Agent Variables](https://docs.tryllengine.com/concepts/agent-variables.html) store — name, type, and initial value, passed to `CreateAgent`. A `TryllAgentComponent.VariableOverrides` entry may override a declared variable's initial value per-instance without changing this asset (Material-Instance pattern).

______________________________________________________________________

## Programmatic alternative

If you do not need a persistent asset, build a `TryllGraphDescription` in C# and create the agent directly from it — no component required:

```
var graph = new TryllGraphBuilder()
    .AddGenerate("gen", new TryllGenerateParams
    {
        // DefaultExit is "" (END) by default
    })
    .SetStartNode("gen")
    .SetDefaultModelName("my-model")
    .Build();

var (agent, error) = await TryllClient.Instance.RequestCreateAgentAsync(graph);
```

Inline graphs are temporary

`TryllAgentComponent.InlineGraphDescription` — assigning a code-built graph to the component instead of using a Workflow Asset — is a temporary feature and will be removed in a future release. When you use `TryllAgentComponent`, author the graph as a Workflow Asset and assign it to the component's **Workflow Asset** field.

______________________________________________________________________

## See also

- [`TryllAgentComponent`](https://docs.tryllengine.com/reference/client-api/unity/tryll-agent-component.html) — uses this asset at agent creation
- [`TryllGraphBuilder`](https://docs.tryllengine.com/reference/client-api/unity/index.html#tryllgraphbuilder) — fluent builder for `TryllGraphDescription`
- [Build a chat agent with a graph](https://docs.tryllengine.com/how-to/build-chat-agent-with-graph.html)
- [Workflows, graphs, nodes](https://docs.tryllengine.com/concepts/workflows-graphs-nodes.html)
- [Agent Variables](https://docs.tryllengine.com/concepts/agent-variables.html)
