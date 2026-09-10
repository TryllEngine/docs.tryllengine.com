# Turn Inspector

The **turn inspector** is the panel that explains a single turn: which nodes ran, how long each took, what prompt the model actually received, what it produced, and what the turn retrieved or called along the way.

It is **one component with three hosts**. The same blocks, in the same order, with the same behaviour, appear in:

| Host                                                                                | Where                         | How you select a turn     |
| ----------------------------------------------------------------------------------- | ----------------------------- | ------------------------- |
| [Chat Window](https://docs.tryllengine.com/v0.8/how-to/test-agent-in-editor.html)   | `Window ▸ Tryll ▸ Chat`       | Click an assistant bubble |
| [Agent Log](https://docs.tryllengine.com/v0.8/how-to/use-agent-log.html)            | `Window ▸ Tryll ▸ Agent Log`  | Click a received turn row |
| [Dialog Lab](https://docs.tryllengine.com/v0.8/how-to/compare-dialog-variants.html) | `Window ▸ Tryll ▸ Dialog Lab` | Click a grid cell         |

This page is the reference for the panel itself. Each host page covers only what is specific to that host — how turns get selected there, and what else that window does.

Prerequisite: diagnostics must be on

The inspector renders the `debug_info` payload a turn carries, which requires `enable_diagnostics` at `CreateAgent`. That flag is **structural** — it cannot be turned on for an agent that already exists.

You do not normally set it yourself. Chat enables it for the agent it creates, and the Agent Log holds a diagnostics-demand lease so agents created while it is open request diagnostics too. Neither writes to your scene or assets. The consequence is that the Agent Log must be **open before an agent is created** to get depth on that agent's turns — see [Use the Agent Log](https://docs.tryllengine.com/v0.8/how-to/use-agent-log.html#open-before-you-create-agents).

______________________________________________________________________

## The one thing to understand first: everything is node-attributed

A Tryll turn is a graph run, not a single model call. A turn can run a classifier, then a retriever, then a generator — each with its own parameters, its own prompt, and its own output.

So every block below the Route breadcrumb reads **per node execution**, not per turn. A graph with two `Retrieve` nodes shows two hit tables. A graph with a classifier and a generator shows two prompt subsections, each with its own heading (`#2 answer (GenerateNode) [default]`).

**Click a node chip in the Route breadcrumb to narrow the entire panel to that one node** — its parameters, its prompt, its retrieval hits. Click the same chip again to go back to the whole turn. This is the fastest way to answer "what exactly did the classifier see?" in a multi-node graph, and it is the primary navigation control of the panel.

The one block node scope does **not** narrow is Raw JSON, which always shows the whole turn.

______________________________________________________________________

## Block reference

| Block               | Shown when                                                  | Starts    | Copy |
| ------------------- | ----------------------------------------------------------- | --------- | ---- |
| **Route**           | Always                                                      | Expanded  | —    |
| **Timing**          | Always                                                      | Expanded  | —    |
| **Parameters**      | Diagnostics present                                         | Collapsed | —    |
| **Checks**          | Dialog Lab only, with authored checks                       | Expanded  | —    |
| **Prompt & output** | A node produced a prompt or output                          | Expanded  | ✅   |
| **Retrieval**       | The turn ran a `Retrieve` node                              | Expanded  | —    |
| **Intent**          | The turn ran a `ClassifyIntent` or `ClassifyIntentLLM` node | Expanded  | —    |
| **Tool calls**      | The turn ran a `ToolCall` node                              | Expanded  | —    |
| **Errors**          | The turn ended in error or cancellation                     | Expanded  | —    |
| **Raw JSON**        | Always                                                      | Collapsed | ✅   |

A block with nothing to say is **hidden, not rendered empty**. If you do not see Retrieval, the turn did not retrieve.

### Route

The execution order, as a breadcrumb of `node name [exit taken]` chips — one chip per node *execution*, so a node entered twice appears twice.

Chips are **selectable**: click one to scope the whole panel to that node, click it again to clear the scope. The scoped chip is highlighted and the Route header's trailing text changes to `<node> only`.

"Other branch" exits (`not_found`, `triggered`, `no_tool_called`) are coloured distinctly from errors — they are usually correct behaviour, not failures.

If the turn never executed, the block says so rather than rendering an empty breadcrumb.

### Timing

A stacked bar of per-node duration, plus the turn total in the header.

- Each node gets a segment sized by its own `duration_s`. Nodes that took no measurable time render grey at zero width — a model-free node like `Branch` or `Transform` normally does.
- **Time to first token** is marked separately: the delay before the first token arrived, which is the number that governs perceived responsiveness.
- Pause-related figures (`paused_duration_s`, `pause_count`) appear for turns that paused.

### Parameters

The parameters each node actually ran with, dumped from the turn's `diagnostics.parameters` grouped under a heading per node.

This is **the parameters the server records into diagnostics, not every field on the node**. Some node fields are deliberately not recorded. When a value is an array — a `ToolCall` node's `tools` signatures, for instance — the elements stack down the value column rather than collapsing to a bare count.

Starts collapsed, because on most turns it restates configuration you already know.

### Checks

**Dialog Lab only.** The authored deterministic checks for that step — Must Contain, Must Not Contain, Regex Forbid, Max Words — with pass/fail per check. The header shows `N failed` or `all passed`.

The other two hosts have no authored checks, so the block is absent there rather than empty.

### Prompt & output

The block most often worth opening. For each node that prompted a model, it shows the **rendered prompt in the form actually sent** — system, user and assistant messages as the model received them — followed by what that node generated.

This is where projection becomes visible: history replay, `{{#instructions}}` blocks, knowledge components and Mustache slot substitution have all already happened in what you see here. If a prompt does not contain what you expected, this block is where that shows up.

**Copy** on the header lifts the transcript to the clipboard without expanding the section. When a node scope is active it copies the scoped node only, matching what is on screen.

Dialog Lab: two-sided prompt diff

Pinning two cells in the Lab (⇧-click, or the ⊙ in a column header) turns this block into a line diff — green `+` for lines only in the selected cell, red `−` for lines only in the other. The red side is usually the interesting one: it shows what a variant *dropped*. Each node is diffed against its own counterpart, and a node that ran on only one side is called out as such.

### Retrieval

One subsection per `Retrieve` node execution. The header is a funnel — `input.query` → `raw_result_count` → `filtered_count` → `result_count` — with `threshold`, `top_k` and the storage name.

The body is the hit table: each retrieved chunk with its distance, in server order (which for Hybrid retrieval is RRF rank). Rows beyond the threshold are dimmed rather than removed, so you can see the near-misses that did not make it in. A null distance renders as `—`.

See [Retrieve](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve.html) and [Embedded String Storage](https://docs.tryllengine.com/v0.8/reference/embedded-string-storage.html).

### Intent

One subsection per `ClassifyIntent` or `ClassifyIntentLLM` execution in scope. The header summarises the decision — `ask_wing · found`, or `not_found · below_threshold` — and a `found` exit is coloured as success, `not_found` as a warning. `not_found` is usually correct behaviour, not a failure.

The block is **hidden when the turn ran no classifier**, including Generate-only graphs. `IntentToInstruction` is not a classifier: it is a lookup, and its result stays in Parameters. Click a Route chip to narrow the block to that one execution.

**Embedding** (`ClassifyIntent`). A muted funnel line (`query → storage · metadata_field · threshold`), then either a winner card (intent, distance, id, index, record text) or a rejected pick (`rejected ask_wing @ 0.3120 (threshold 0.25)`). A top-k table lists ranked candidates; rows beyond the threshold are dimmed rather than removed. A null distance renders as `—`.

**LLM** (`ClassifyIntentLLM`). A muted line (`model · threshold · margin · history_turns`), a horizontal probability bar per candidate (sorted by probability, ties keep `intents_ids` order), and the `predicted_letter` / `top_prob` / `second_prob` / `margin_result` strip. The winning or rejected label is shown in words — you do not map the letter by hand. A `probs` / `intents_ids` length mismatch draws a warning and shows only the common prefix.

When the turn's interaction components are present, a footer lists the labels that were actually attached: `attached to dialog: ask_wing`. When `include_interaction_in_diagnostics` is off, that footer is omitted silently. Unlike [Tool calls](#tool-calls), this block does not depend on interaction diagnostics — the classification itself is always in `output.classification`.

`notify_client` does not gate this block. A classifier with `notify_client` left at `Disabled` still appears here; it simply produces no Agent Log event. See [Classify Intent](https://docs.tryllengine.com/v0.8/reference/nodes/classify-intent.html) and [Classify Intent (LLM)](https://docs.tryllengine.com/v0.8/reference/nodes/classify-intent-llm.html).

### Tool calls

For each `ToolCall` node execution: which function the model called, the arguments it passed, and the result that came back — as JSON. The header shows the call count and the exit taken.

Two things gate what appears here, and both are configuration rather than defects:

- **The node's `disposition`.** Whether a result is returned to the model at all, and whether the client is notified, is decided by `ToolCallParams.disposition`. A disposition that does not return results leaves the result side empty. See [Tool Call](https://docs.tryllengine.com/v0.8/reference/nodes/tool-call.html).
- **Server config `include_interaction_in_diagnostics`.** Tool-call results are joined through the turn's interaction components. With this off, the panel explains that rather than silently rendering a short list.

### Errors

Present only for turns that ended in error or cancellation, carrying the terminal status and the error detail. The header shows the status.

### Raw JSON

The escape hatch: the complete `debug_info` payload the turn carried, pretty-printed. Everything above is a purpose-built view over this — so anything the panel has no dedicated block for is still reachable here.

The payload is shown **verbatim**: the editors add no keys, rename none, and drop none. Every field in it is documented in [Turn Diagnostics JSON](https://docs.tryllengine.com/v0.8/reference/turn-diagnostics-json.html), which is also the reference for the same payload as it arrives on the wire in `TurnComplete.debug_info`.

Unlike every other block, Raw JSON ignores the Route breadcrumb's node scope — it always shows the whole turn. **Copy** on the header puts the formatted JSON on the clipboard.

Two things currently live here and nowhere else:

- **Variable replacements and engine/scheduler counters**, including the KV-reuse figures that often explain why one sample was slower than another.

______________________________________________________________________

## When a block is missing or the panel is thin

The panel distinguishes these cases rather than rendering an empty section. If you see less than you expected, it is one of:

| What you see                         | Why                                                                                                           |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| "Created before diagnostics capture" | The agent existed before the Agent Log opened. Recreate it, or create new agents while the window is open.    |
| "Unsupported schema"                 | The turn's `debug_info` is not `schema_version` 2. Raw JSON may still be offered.                             |
| "Turn was not executed"              | Dialog Lab: a seeded turn (injected without running the graph), or a turn after an error in the same column.  |
| A block simply absent                | The turn did not do that thing — no `Retrieve` node ran, no tool was called, no error occurred.               |
| Tool-call results empty              | The node's `disposition` does not return results, or `include_interaction_in_diagnostics` is off server-side. |

______________________________________________________________________

## Host differences

The panel is the same in all three windows and in both engines. What differs is its framing:

|                       | Chat                                         | Agent Log                                                 | Dialog Lab |
| --------------------- | -------------------------------------------- | --------------------------------------------------------- | ---------- |
| Default visibility    | Visible                                      | Collapsed                                                 | Visible    |
| Width persisted       | Per window, independently of the other hosts | Per window                                                | Per window |
| Checks block          | —                                            | —                                                         | ✅         |
| Prompt diff (pinning) | —                                            | —                                                         | ✅         |
| Non-turn rows         | —                                            | Sent / system rows use a **different**, simpler inspector | —          |

In the Agent Log, only *received turn* rows open this panel. Sent rows and system rows are selectable too, but they open simpler panels showing the wire payload, deferred variable batches, and the server's `Ack` or error — see [Use the Agent Log](https://docs.tryllengine.com/v0.8/how-to/use-agent-log.html#the-inspector-changes-with-the-row).

Unity and Unreal parity

Both engines implement the same ten blocks in the same order. Chrome — fonts, splitter behaviour, scrollbar feel — differs, and that is expected.

______________________________________________________________________

## Related

- [How-to: Test an Agent in the Editor](https://docs.tryllengine.com/v0.8/how-to/test-agent-in-editor.html) — the Chat Window
- [How-to: Use the Agent Log](https://docs.tryllengine.com/v0.8/how-to/use-agent-log.html) — per-agent send/receive narrative
- [How-to: Compare Dialog Variants](https://docs.tryllengine.com/v0.8/how-to/compare-dialog-variants.html) — Dialog Lab
- [Turn Diagnostics JSON](https://docs.tryllengine.com/v0.8/reference/turn-diagnostics-json.html) — every field of the payload this panel renders
- [Concepts: Projection and token budgets](https://docs.tryllengine.com/v0.8/concepts/projection-and-token-budgets.html) — why the rendered prompt looks the way it does
- [Concepts: Slots and inter-node value passing](https://docs.tryllengine.com/v0.8/concepts/slots-and-inter-node-value-passing.html) — how nodes hand values to each other
- [Reference: Agent Parameters](https://docs.tryllengine.com/v0.8/reference/agent-parameters.html) — `enable_diagnostics` and the rest of `CreateAgentRequest`
