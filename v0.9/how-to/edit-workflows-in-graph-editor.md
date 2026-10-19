# Edit Workflows in the Graph Editor

The Unity plugin includes an editor-only **Workflow Graph** window that renders a [`TryllWorkflowAsset`](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-workflow-asset.html) as a node graph: each node is a box, each exit is an output port, and the wiring you would otherwise type as node names becomes draggable edges. It is a view over the same asset you edit in the Inspector — there is no second file and no separate save format.

This is the Unity graph editor

This page covers the **Unity** Workflow Graph window. Unreal ships its own visual workflow graph editor — see [Edit Workflows in the Unreal Graph Editor](https://docs.tryllengine.com/v0.9/how-to/edit-workflows-in-unreal-graph-editor.html). You can also author graphs without a visual editor through the `UTryllWorkflowAsset`/`TryllWorkflowAsset` Details panel, the C++ graph builder API, or (Unreal) Blueprint — see [Build a Chat Agent with a Graph](https://docs.tryllengine.com/v0.9/how-to/build-chat-agent-with-graph.html).

Prerequisites

- The Tryll Unity package installed in your project. If not, see [First Inference in Unity](https://docs.tryllengine.com/v0.9/getting-started/first-inference-unity.html).
- A `TryllWorkflowAsset` to open. Create one with **Assets → Create → Tryll → Workflow Asset**.

## Step 1 — open the window

Either:

- **Window → Tryll → Workflow Graph**, then drag a `TryllWorkflowAsset` into the **Workflow** field at the top; or
- **Double-click** a `TryllWorkflowAsset` in the Project window.

The window docks like any Unity editor panel. The canvas fills the left side; a property panel sits on the right, and the divider between them is draggable to resize.

## Step 2 — read the graph

| Element          | Meaning                                                                                                                              |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| **Entry** node   | The graph's start. Its single edge points at the `StartNode`.                                                                        |
| Node box         | One `TryllNodeDescription`. The title is the node **Name**; the subtitle is the node type (Generate, Retrieve, …).                   |
| Left **in** port | The node as a routing target. Accepts any number of incoming edges.                                                                  |
| Right ports      | One per [exit](https://docs.tryllengine.com/v0.9/reference/nodes/index.html) the node declares (`default`, `found`, `not_found`, …). |
| Edge             | A wired exit. The source exit's target is the node it points at.                                                                     |
| Unconnected exit | **END** — the turn finishes after that node.                                                                                         |

Node positions are laid out automatically each time you open the asset (left to right from Entry); positions are not saved.

## Step 3 — wire and unwire

- **Wire an exit:** drag from a node's right-hand exit port to another node's **in** port. This writes the target node's name into that exit field.
- **Unwire:** select the edge and press **Delete**. The exit reverts to **END** (empty).
- **Set the start node:** drag the **Entry** node's port to any node. Deleting that edge clears `StartNode`.

Each exit holds exactly one target, so an exit port allows only one outgoing edge — to re-point it, draw the new edge (the old one is replaced). Cycles are allowed; the graph is a router, not a DAG.

## Step 4 — add and remove nodes

- **Add:** right-click the canvas → **Add Node →** pick a type. A new node is created with a unique name and default parameters, placed at the cursor. Rename it and fill in its parameters in the property panel (Step 5).
- **Remove:** select a node and press **Delete**. Any exits on *other* nodes that pointed at it are cleared to END automatically, and `StartNode` is cleared if it named the deleted node. The Entry node cannot be deleted.

## Step 5 — edit node parameters

Click a node to select it. The property panel on the right binds to that node's `Params` and renders the **same drawers** the Inspector uses — model-name dropdowns, exit dropdowns, `[Range]` sliders, storage file pickers, multi-line text. Editing here changes the asset directly.

Canvas and panel share one source of truth

Both the canvas and the property panel edit the one `TryllWorkflowAsset`. If you change an exit *in the panel's dropdown*, the canvas edge updates after you click **Reload** (the panel uses Unity's own binding path, separate from the canvas redraw).

### Slot inputs

Nodes pass values to each other through named **slots** (see [Slots and inter-node value passing](https://docs.tryllengine.com/v0.9/concepts/slots-and-inter-node-value-passing.html)). On a node that consumes a slot — `Generate`, `GenerateAndSpeak`, `Retrieve`, `Speak` — the **Input** field is a **dropdown** of the slots available at that point in the graph (each producing node's `Output Name`, or its node name if unset, plus the reserved `user_message`). Leaving **Input** empty means `user_message`.

Renaming a node propagates automatically: when you rename a producing node, the editor rewrites every downstream node's **Input** (and any `{{slot.<name>}}` reference in a template) to the new name, so wiring by slot survives renames.

## Step 6 — validate

The window runs the same checks as **Assets → Tryll → Validate Workflow** after every edit and lists findings above the canvas:

- **Errors** (red): missing `StartNode`, duplicate node names, an exit that points at a node that does not exist.
- **Warnings** (yellow): orphan nodes (no incoming edge and not the start), storage-path issues.

Fix errors before you ship the asset to an agent — the server reads the exit strings at `CreateAgent` time.

## Undo, redo, and save

Edits go through Unity's standard undo system: **Ctrl+Z** / **Ctrl+Y** revert and replay graph and parameter changes, and they interleave with edits you make in the normal Inspector (one shared history). Save the asset with **Ctrl+S** like any Unity asset.

## Common problems

**The window is empty** : No asset is assigned. Drag a `TryllWorkflowAsset` into the **Workflow** field, or double-click one in the Project window.

**An exit port won't start a new edge** : That exit already has a target (exits are single-target). Draw the replacement edge directly, or delete the existing edge first.

**An edge I drew in the panel's dropdown isn't on the canvas** : Click **Reload**. Dropdown edits and canvas edits share the asset but the canvas only redraws on open/reload/undo.

## Related

- [How-to: Build a Chat Agent with a Graph](https://docs.tryllengine.com/v0.9/how-to/build-chat-agent-with-graph.html)
- [How-to: Test an Agent in the Editor](https://docs.tryllengine.com/v0.9/how-to/test-agent-in-editor.html)
- [Concept: Workflows, graphs, nodes](https://docs.tryllengine.com/v0.9/concepts/workflows-graphs-nodes.html)
- [Reference: Workflow Nodes](https://docs.tryllengine.com/v0.9/reference/nodes/index.html)
- [Reference: TryllWorkflowAsset (Unity)](https://docs.tryllengine.com/v0.9/reference/client-api/unity/tryll-workflow-asset.html)
