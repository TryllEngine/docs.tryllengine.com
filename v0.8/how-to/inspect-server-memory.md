# Inspect Server Memory

The editor **Memory** window (Unity and Unreal) shows how much RAM and VRAM the Tryll server process is using right now — models, per-agent KV caches, and the remainder we cannot attribute. It is the live check for the recipe in [Estimate Memory Footprint](https://docs.tryllengine.com/v0.8/how-to/estimate-memory-footprint.html).

It answers questions a download-size spreadsheet cannot: *is that model actually resident?*, *how much KV is sitting on background agents?*, *is "other" on the GPU the editor, or us?*

Scope: the whole server process

The snapshot is **server-wide**. Every session on that process is in the numbers — Chat, Model Manager, Play-in-Editor, a Python client, the QA harness. That is the opposite of the [Agent Log](https://docs.tryllengine.com/v0.8/how-to/use-agent-log.html), which sees only this editor connection. A row you do not recognise is not a bug in the window.

Prerequisites

- The Tryll Unity or Unreal plugin installed.
- A Ready editor session — Chat, Model Manager, PIE, or this window's own auto-launch. You do **not** need an agent, and this window does **not** wait for registered models to download.

## Open the window

| Editor | Menu                              |
| ------ | --------------------------------- |
| Unity  | **Window → Tryll → Memory**       |
| Unreal | **Window → Tryll → Tryll Memory** |

The window reuses the shared editor session. If Chat or Model Manager is already Ready, **Refresh** is enabled immediately. If not, it auto-launches the bundled server and calls `CreateSession` the same way Model Manager does — turn on **Auto Launch Server** in Tryll runtime settings if you usually rely on that.

Closing the window releases the connection lease. It does **not** kill the server.

Disconnected state still shows this editor process's RAM. The label is **Unity reserved** in Unity (allocator reserved) and **Unreal process RAM, not editor VRAM** in Unreal (`FPlatformMemory` used physical). Those are different metrics — do not compare them to each other. The server tables stay on screen if you already have a snapshot; they are not wiped.

## Take a snapshot

There is no poll and no graph. Click **Refresh**.

Refresh is disabled until the session is Ready, and while a fetch is in flight. A late reply cannot overwrite a newer one. If a fetch fails after you already have numbers, a stale-snapshot banner keeps the last good table and shows the error.

## How to read the numbers

### Confidence tiers

Every component row has a visible **Confidence** column. That is the feature: an estimate must never look like a measurement.

| Tier          | Meaning                                                                          | How it renders                                                                                      |
| ------------- | -------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| **Measured**  | The engine or OS told us this number                                             | Plain bytes                                                                                         |
| **Derived**   | Computed from measured inputs (for example device `other` = total − free − ours) | Plain bytes                                                                                         |
| **Estimated** | A calibrated band, not a reading                                                 | Leading `~`, and a high band when the collector supplied one. Hover the row for the estimate basis. |

### System — working set, private, unattributed

The system table is four rows: this editor process's RAM, **Working set**, **Private**, **Unattributed**.

| Field                                       | What it is                                                                                                   |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Unity reserved** / **Unreal process RAM** | This editor process — RAM, not editor VRAM. Unity reports allocator reserved; Unreal reports `UsedPhysical`. |
| **Working set**                             | Pages the OS currently has resident for `tryll_server`                                                       |
| **Private**                                 | Commit that is not shareable — closer to "what we allocated"                                                 |
| **Unattributed**                            | Working set minus the RAM we assigned to components                                                          |

Memory-mapped model files count as resident pages but are not allocations we make. A VRAM-placed model therefore often shows up in **unattributed** rather than under its own RAM column. The window calls that out when unattributed is more than half the working set and a model is resident.

### Devices — ours / other / free

One table row per device: **Device**, **Kind**, **Ours** (model weights + KV + compute), **Other** (everything on the device that is not Tryll), **Free**, **Total**.

`other` is where the editor, the compositor, and other processes live. Do not treat it as a Tryll leak. The editor's own VRAM cannot be split out of that number.

If the backend is not initialised yet, devices register on the first model load. That is not an error.

### KV by agent — reserved vs context used

The KV table is sorted **background → buffered → visible**, then by reserved bytes descending. Background KV is the reclaim signal: nobody is waiting on that memory.

| Column           | How to read it                                                                                                                                                                                                |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Class**        | Scheduler class of the last turn (`visible` / `buffered` / `background`), plus `evicted` when the context is not resident. Describes the work, not the agent.                                                 |
| **When**         | Relative time, or **never run** if the context has never completed a turn. Idle class and "never run" are different: a context can be allocated and unused.                                                   |
| **Context used** | A bar plus `used / capacity (pct)`. Headroom to the limit, **not memory growth**. The KV buffer is allocated for the full `context_size` when the context is created and does not grow with the conversation. |
| **Reserved**     | Bytes actually held for that context (RAM + VRAM)                                                                                                                                                             |

### Components

Kind, name, confidence, RAM (a `~` band when estimated), VRAM. KV rows are listed in the agent table above, not repeated here.

## What changes these numbers

| Knob                                               | Where                                                                                                          |
| -------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Load / unload / pin                                | [Manage Models in the Editor](https://docs.tryllengine.com/v0.8/how-to/manage-models-in-editor.html)           |
| `MemoryPlacement` (RAM vs VRAM)                    | [Choose where models run](https://docs.tryllengine.com/v0.8/how-to/choose-where-models-run.html)               |
| Per-node `context_size` / `context`                | [Run many agents on a small GPU](https://docs.tryllengine.com/v0.8/how-to/run-many-agents-on-a-small-gpu.html) |
| Prefill / Evict / `DeferAllocation`                | [Manage an agent's KV cache](https://docs.tryllengine.com/v0.8/how-to/manage-agent-kv-cache.html)              |
| Server `engines.llama_cpp.inference.default_n_ctx` | [Server configuration](https://docs.tryllengine.com/v0.8/reference/server-configuration.html)                  |
| Catalog `kv_cache_type`                            | [Model management](https://docs.tryllengine.com/v0.8/reference/model-management.html)                          |

## Relationship to the server monitor

|           | Memory window                            | Server monitor Memory panel      |
| --------- | ---------------------------------------- | -------------------------------- |
| Scope     | Whole server process                     | Whole server process             |
| Collector | The same `CollectMemorySnapshot`         | The same `CollectMemorySnapshot` |
| Surface   | Unity / Unreal editor, over the TCP wire | Browser, `GET /api/memory`       |
| Refresh   | Button only                              | SSE / poll, plus a sparkline     |

The numbers should match. The monitor still has the sparkline and stacked bars; this window does not.

C++ (`TryllClient::GetMemoryConsumption`), Python (`client.get_memory_consumption()`), Unity (`RequestGetMemoryConsumptionAsync`), and Unreal (`UTryllSubsystem::RequestGetMemoryConsumption`) return the same snapshot.

## Related

- [Estimate Memory Footprint](https://docs.tryllengine.com/v0.8/how-to/estimate-memory-footprint.html) — the pre-ship recipe this window checks live
- [Use the Agent Log](https://docs.tryllengine.com/v0.8/how-to/use-agent-log.html) — per-connection traffic, not process memory
- [Wire Protocol](https://docs.tryllengine.com/v0.8/reference/wire-protocol.html) — `GetMemoryConsumptionRequest` / `Response`
- [Unity `TryllClient`](https://docs.tryllengine.com/v0.8/reference/client-api/unity/tryll-client.html) — `RequestGetMemoryConsumptionAsync`
- [Unreal C++ API](https://docs.tryllengine.com/v0.8/reference/client-api/unreal/cpp/index.html) — `RequestGetMemoryConsumption` / `FTryllMemorySnapshot`
