# Edit Workflows in the Unreal Graph Editor

The Unreal plugin ships a visual **workflow graph editor** for `UTryllWorkflowAsset`, mirroring the Unity graph window. Double-click a workflow asset and you get a node canvas: each node is a workflow step, each wire is an exit route, and an **Entry** node marks where a turn starts. It is the same `FTryllGraphDescription` underneath — the graph editor is just a visual front end for the data you would otherwise hand-edit in the Details panel.

Prerequisites

- The Tryll plugin installed in your project — see [First Inference in Unreal](https://docs.tryllengine.com/v0.9/getting-started/first-inference-unreal.html).
- A `UTryllWorkflowAsset`. Create one from the Content Browser: **Add → Tryll → Workflow Asset**.

## Step 1 — open the editor

Double-click the `UTryllWorkflowAsset` in the Content Browser. The asset editor opens with two panels:

| Panel               | Contents                                                                                        |
| ------------------- | ----------------------------------------------------------------------------------------------- |
| **Graph** (centre)  | The node canvas. Pan with right-drag, zoom with the mouse wheel, marquee-select with left-drag. |
| **Details** (right) | Parameters for the currently selected node.                                                     |

A toolbar across the top adds **Validate** and **Auto Layout** buttons and a live error/warning count, alongside the usual **Save**.

## Step 2 — add nodes

Right-click an empty spot on the canvas and pick a type from the **Add Node** menu (Generate, ToolCall, Retrieve, CannedResponse, …). The menu lists every node type the server supports — it is generated from the schema, so new node types appear automatically.

You can also **drag off a pin** into empty space and release: the same menu appears, and the new node is auto-wired to the pin you dragged from.

Each new node gets a unique name (`generate1`, `toolcall1`, …). To rename one, either double-click its title on the canvas or edit **Node Name** in the **Details** panel — both routes apply the same rules:

- names must be letters, digits and underscores only, and cannot start with a digit; `user_message` is reserved for the implicit input slot. An invalid name is refused with an inline message instead of being committed.
- a name already used by another node gets the smallest free numeric suffix (renaming a second node to `refine` yields `refine2`), so two nodes can never end up sharing a name.

## Step 3 — wire exits

Every node has one **input** pin on the left and one **output** pin per exit on the right. Drag from an output pin to another node's input pin to route that exit there.

- An exit left **unconnected** means **END** — the turn finishes after that node.
- Output exits are **single-target**: dragging a new wire from an exit replaces the old one.
- Wire the **Entry** node's pin to whichever node should run first — that sets the workflow's start node.

Exit names read as friendly labels (`Tool Called`, `No Tool Called`) but map to the wire exits (`tool_called`, `no_tool_called`) under the hood. Because routing lives on the wires, you can rename a node freely — inbound wires follow it.

## Step 4 — edit node parameters

Select a node to edit its parameters in the **Details** panel — **Node Name** first, then system prompt, model, sampling, storage paths, and so on, with the same dropdowns and pickers as the regular Details view. Exit fields are intentionally **hidden here**: you set routing by dragging wires, not by typing node names.

Select the **Entry** node to set the workflow's **Default Model Name** — the fallback model for any `Generate`-family node that does not specify its own.

### Slot inputs

Nodes pass values to each other through named **slots** (see [Slots and inter-node value passing](https://docs.tryllengine.com/v0.9/concepts/slots-and-inter-node-value-passing.html)). On a slot-consuming node — `Generate`, `GenerateAndSpeak`, `Retrieve`, `Speak` — tick **Override Input** and pick the source slot from the **Input** dropdown: it lists the slots available upstream (each producing node's **Output Name**, or its node name if unset, plus the reserved `user_message`). Left unset, **Input** resolves to `user_message`.

Renaming a node propagates automatically: renaming a producing node rewrites every downstream node's **Input** and any `{{slot.<name>}}` template reference to the new name, so slot wiring survives renames.

## Step 5 — validate

The editor re-checks the graph after every edit and flags problems inline:

- a node with a problem shows an **error (red)** or **warning (yellow)** badge with a tooltip describing the issue;
- the toolbar shows the total **error / warning** count.

Checks include: a missing or unresolved start node, duplicate node names, an exit pointing at a node that no longer exists, unreachable (orphan) nodes, and storage paths that are not valid relative paths. Click **Validate** to re-run on demand.

## Step 6 — arrange and save

Nodes remember where you put them — drag them to taste and the layout persists with the asset. **Auto Layout** re-arranges everything left-to-right from the Entry node (handy for an imported or messy graph).

Press **Save** (or **Ctrl+S**). The graph is written back into the asset's `FTryllGraphDescription`; assign that asset to a `UTryllAgentComponent`'s **Workflow Asset** to run it, or test it directly in the [Chat Window](https://docs.tryllengine.com/v0.9/how-to/test-agent-in-editor.html).

## Common problems

**Double-click opens a plain Details view, not the graph** : The editor module did not load. Confirm the **Tryll Client** plugin is enabled and you are running the Editor build of your project.

**"StartNode is empty" warning** : Nothing is wired to the **Entry** node. Drag the Entry pin to your first node.

**A node shows "exit … references unknown node"** : This only happens if the description was edited outside the graph editor. Open the node, re-drag the exit, and save — the editor rewrites the routing from the wires.

## Related

- [How-to: Build a Chat Agent with a Graph](https://docs.tryllengine.com/v0.9/how-to/build-chat-agent-with-graph.html)
- [How-to: Test an Agent in the Editor](https://docs.tryllengine.com/v0.9/how-to/test-agent-in-editor.html)
- [Concept: Workflows, Graphs, and Nodes](https://docs.tryllengine.com/v0.9/concepts/workflows-graphs-nodes.html)
- [Reference: Workflow Nodes](https://docs.tryllengine.com/v0.9/reference/nodes/index.html)
