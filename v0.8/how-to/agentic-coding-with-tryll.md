# Agentic Coding With Tryll

AI coding agents such as Claude Code, Cursor and Copilot can build Tryll agents for you: author the workflow, wire it into your game, and test it. This page covers how to set them up for Tryll and how to brief them so they deliver a working result.

## Start every session with `AGENTS.md`

Both plugins ship an **`AGENTS.md`** file at their root — in the TryllClient plugin folder for Unreal, and at the package root for Unity. It gives a coding agent the essentials in one read:

- how Tryll works — an agent is a graph of nodes, not a single prompt;
- where Tryll differs from cloud LLM SDKs, which is where coding agents most often go wrong;
- links to the pages in these docs that cover each topic in depth.

Instruct the agent to read `AGENTS.md` before it writes any code or assets.

## Unreal

### Why use MCP

Much of an Unreal project lives in binary assets — levels, Blueprints and Tryll workflow assets — that a coding agent cannot edit as text. Epic's **Model Context Protocol** (MCP) plugin, available as an experimental plugin in UE 5.8, lets the agent work directly in the running editor instead.

For Tryll workflows, install the **Tryll MCP** plugin alongside it. It gives the agent dedicated tools for creating and editing workflow assets. Epic's general-purpose editing tools are not designed for Tryll's workflow format and should not be used to write workflows.

### Requirements

- Unreal Engine 5.8. Tryll MCP depends on Epic's experimental **Toolset Registry** plugin, which is not available in 5.7. The TryllClient plugin itself supports both 5.7 and 5.8.
- The TryllClient plugin, set up as described in [First inference (Unreal)](https://docs.tryllengine.com/v0.8/getting-started/first-inference-unreal.html).

### Set up

1. Copy the `TryllMCP` folder from the Tryll release into your project's `Plugins/` folder, next to `TryllClient`.

1. In **Edit > Plugins**, enable **Tryll MCP**, **Model Context Protocol** and **All Toolsets**, then restart the editor. **All Toolsets** provides Epic's tools for levels, assets and Blueprints.

1. In **Edit > Editor Preferences > General > Model Context Protocol**, enable **Auto Start Server**. The server then starts with the editor, at `http://127.0.0.1:8000/mcp` by default.

1. Connect your coding agent to that address. For Claude Code, add a `.mcp.json` file at the project root:

   ```
   {
     "mcpServers": {
       "unreal-mcp": {
         "type": "http",
         "url": "http://127.0.0.1:8000/mcp"
       }
     }
   }
   ```

1. Open the editor first, then start the agent session. An agent session started before the MCP server is running will not see the Unreal tools; if that happens, restart the session.

Only one editor can use port 8000 at a time. If the server fails to start, the Output Log shows `HttpListener unable to bind to 127.0.0.1:8000`. Close any other open editor, then restart this one.

### Authoring workflows

Tryll MCP provides four tools, grouped under `TryllMCP.TryllWorkflowToolset`:

| Tool                | Purpose                                                                                                                                                                                         |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `DescribeNodeTypes` | Lists the available node types and their parameters, including value ranges, allowed values, and which parameters are exits. Pass a single type, such as `Generate`, to describe only that one. |
| `ValidateWorkflow`  | Checks a workflow without changing the project, and reports every problem in one response.                                                                                                      |
| `WriteWorkflow`     | Creates or replaces a workflow asset and saves it. A workflow with errors is rejected and the existing asset is left unchanged.                                                                 |
| `ReadWorkflow`      | Returns an existing workflow in the same format `WriteWorkflow` accepts.                                                                                                                        |

A workflow is always written as a whole. To change part of one, the agent reads it, edits it, and writes it back. The format looks like this:

```
{
  "startNode": "guard",
  "variables": [
    { "name": "price", "type": "Int", "allowOutputSubstitution": true, "value": 120 }
  ],
  "nodes": [
    { "name": "guard", "type": "RegexGuardrail",
      "params": { "notTriggeredExit": "haggle" } },
    { "name": "haggle", "type": "Generate",
      "params": { "systemPrompt": "You are a merchant. The fuel cell costs __PRICE__ credits.",
                  "bSubstituteAgentVariables": true } }
  ]
}
```

- Parameter names match those listed by `DescribeNodeTypes`. An unknown parameter name is reported as an error, together with the valid names.
- Parameters you set are applied as given. Parameters you omit keep their default, or are inherited from the model catalog.
- Exits are parameters, such as `defaultExit` or `triggeredExit`, that hold the name of the next node. An empty exit ends the turn.

Recommended sequence for the agent: call `DescribeNodeTypes` for the node types it plans to use, run `ValidateWorkflow` until no errors remain, save with `WriteWorkflow`, then confirm the result with `ReadWorkflow`.

## Unity

No MCP setup is required for Tryll in Unity. Workflows are written in C# with `TryllGraphBuilder`, or stored as `TryllWorkflowAsset` files, both of which coding agents can edit directly. Provide `AGENTS.md` and follow the practices below. Unity's own MCP integration remains useful for scene work, but Tryll does not depend on it.

## Best practices for briefing the agent

These apply in both engines.

- **Scope each task to one agent and one workflow.** Broad requests — several NPCs, a UI and a scene at once — dilute the agent's effort, and most of it goes into the parts that are not Tryll. Build up in steps.
- **Write the acceptance tests into the prompt**, as exact player lines and the expected response. For example: *"Ask the price of a fuel cell: the merchant quotes 120. Offer 90: the merchant counters and does not accept 90."* Concrete tests lead the agent to play the game and check the result, which catches problems that reading the code does not, such as the model repeating an example line from its prompt verbatim.
- **Point the agent at the relevant documentation.** Name the pages for the feature you are building, for example [Keep Merchant Prices Deterministic](https://docs.tryllengine.com/v0.8/how-to/keep-merchant-prices-deterministic.html) or [Substitute Agent Variables in Output](https://docs.tryllengine.com/v0.8/how-to/substitute-agent-variables-in-output.html), and ask it to consult [Workflow Nodes](https://docs.tryllengine.com/v0.8/reference/nodes/index.html) before setting node parameters.
- **Download the models in advance** with the [Model Manager](https://docs.tryllengine.com/v0.8/how-to/manage-models-in-editor.html), so the agent spends its time on the feature itself.
- **Unreal: require the agent to save its work.** Levels and assets exist only in the open editor until they are saved. Ask the agent to save them and confirm the files exist on disk.
- **Unreal: keep asset paths consistent.** Paths are case-sensitive — `/Game/TradingPost` and `/Game/tradingpost` are different locations.

## Related

- [Edit Workflows in the Unreal Graph Editor](https://docs.tryllengine.com/v0.8/how-to/edit-workflows-in-unreal-graph-editor.html) — review or adjust an agent-authored workflow by hand.
- [Test an Agent in the Editor](https://docs.tryllengine.com/v0.8/how-to/test-agent-in-editor.html) — talk to a workflow without entering Play.
