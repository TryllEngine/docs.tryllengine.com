# Run many agents on a small GPU

A game with several NPC agents can run out of video memory long before it runs out of speed. This page covers the two levers that matter, in the order they are worth pulling: **sizing each node's context window**, then **evicting the agents nobody is talking to**.

The figures throughout come from a five-agent murder-mystery demo — four suspects plus a background note-taker, nine language-model contexts in total, on a 16 GB card. Your numbers will differ; the shape of the problem will not.

## The one fact that makes this tractable

**A KV cache is allocated at its full `context_size` the moment the context is created.** It does not grow as the conversation does.

That has three consequences worth internalising before you change anything:

- The cost is **deterministic**. It does not depend on how long anyone plays.
- Short NPC replies do not save memory. Occupancy is irrelevant; the cap is what is allocated.
- You can measure a sizing change in seconds — create the agents, read VRAM, done. No playtest.

In the demo, agent creation added ~1.6 GB in a single step and a full playtest then added under 100 MB more.

## Step 1 — size every LLM node

Every node that owns a language-model context takes a `context_size` in tokens: `Generate`, the LLM half of `GenerateAndSpeak`, `ToolCall`, and `ClassifyIntentLLM`. Left at `0`, **ClassifyIntentLLM** and **ToolCall** use a 2048-token node default. **Generate** and **GenerateAndSpeak** still fall back to the model variant's `context_size`, then the server's `default_n_ctx` — commonly 8192, which is far more than a classifier or tool-call node needs.

The optional `context` table on those nodes is an advanced structural override (`kv_cache_type`, veto-only `offload_kqv`, escape-hatch batch sizes). Leave it unset unless you are tuning a large window; at a right-sized 2048, changing KV dtype rarely saves measurable VRAM.

`context_size` is **structural**: it is fixed when the agent is created and cannot be changed with [Change agent parameters](https://docs.tryllengine.com/v0.8/how-to/change-agent-parameters.html). Set it when you build the graph.

**Bound the prompt, then leave headroom.** For a classifier node the budget is computable: framing prompt + label descriptions + `history_turns` × a turn + the latest player line. A classifier scores one token and generates nothing, so it needs almost no generation reserve. In the demo that came to ~700 tokens worst case, and 2048 was chosen for roughly 3× headroom.

For a `Generate` node, add the system prompt, whatever retrieval or instruction text the template folds in, and a reserve for the reply itself.

Do not size to the exact budget

When the prompt exceeds the window, the token-budget projection trims the **oldest turns**. It keeps working, so the failure is silent: a classifier quietly loses the conversation history it needs to resolve *"And that one?"* or *"Yes, do it."* — the very inputs that are unclassifiable without it. After tightening a classifier's window, check that intents still fire, not merely that the game still runs.

### What a context actually costs

Two parts, and the second is easy to forget:

| Part                                   | Scales with `context_size`?    | Notes                                                          |
| -------------------------------------- | ------------------------------ | -------------------------------------------------------------- |
| KV cache                               | Yes, linearly                  | `2 × layers × kv_heads × head_dim × n_ctx × bytes_per_element` |
| Compute buffers and backend allocation | No — roughly fixed per context | ~145 MB per context in the demo                                |

Two consequences. First, **the same window costs very different amounts on different models** — attention shape matters more than parameter count. In the demo, a 3B classifier with full attention cost ~499 MB at 8192 while a 4B generator cost ~606 MB, and a sliding-window architecture would cost less again.

Do not assume a sliding-window discount

A model with interleaved local attention *might* cap most layers at its window size, which would make a large `context_size` nearly free. Whether the runtime actually does that is a property of the build, not of the architecture on paper. Assuming the discount in the demo under-predicted the saving by 1.78×. Measure one before you plan around it.

Second, once windows are small the **fixed per-context overhead dominates**. At 2048 tokens the demo's classifier contexts were ~125 MB of KV against ~145 MB of buffers — so shrinking the window further buys little, and the lever becomes context *count*, not context *size*.

## Step 2 — keep only the agents in use resident

Sizing is free. Eviction is not: it trades latency for memory, and you should turn it on deliberately.

With every context resident, switching between agents is nearly free. Evicted, the first message to an agent also pays context allocation and a full prefill — on the order of a second or two — in exchange for the memory back. That is the right trade on a GPU that would otherwise not fit, and the wrong one on a GPU with headroom.

### Unity

Add a `TryllKvResidencyPolicy` to any GameObject, register the agents that should participate, and tell it which one the player is using.

```
using Tryll.Client;

_policy = gameObject.AddComponent<TryllKvResidencyPolicy>();
_policy.Enabled     = playerWantsLowVram;  // drive this from a graphics setting
_policy.HotSetSize  = 2;

foreach (var npc in _npcs)
    _policy.Register(npc.AgentComponent, startsResident: !playerWantsLowVram);

// ...whenever the player's focus moves:
_policy.SetActive(selectedNpc.AgentComponent);
```

### Unreal

`UTryllKvResidencyPolicy` is an `ActorComponent` with the same surface, all Blueprint-callable: `Register`, `SetActiveAgent`, `Unregister`, `RestoreAll`, plus `bEnabled`, `HotSetSize` and `PrefillDebounceSeconds` as exposed properties.

### Pair it with deferred allocation

The policy evicts, but most of the saving comes from **never allocating in the first place**. Create the agents it manages with `DeferAllocation`:

```
component.KvCacheInitialization = playerWantsLowVram
    ? TryllKvCacheInitialization.DeferAllocation
    : TryllKvCacheInitialization.AllocateOnly;
```

This is structural too, so it is fixed at creation. Toggling the setting mid-session still evicts and restores correctly, but only a restart gets the full benefit — worth saying in your options UI if the setting is player-facing.

### What the defaults are protecting you from

Three behaviours are easy to omit and unpleasant to debug:

- **The prefill debounce.** A prefill cannot be cancelled once started, so the only way to avoid paying for agents the player merely scrolled past is not to start. The policy waits (0.35 s by default) for a selection to settle.
- **A hot set larger than one.** "Go back to the one I was just using" is the commonest interaction there is. `HotSetSize = 1` maximises the saving and punishes exactly that.
- **Idle-only operations.** Prefill and evict answer `AgentBusy` against a running turn. The policy checks first and treats `AgentBusy` as a retry rather than an error.

Correctness never depends on this

An evicted context is restored automatically before the next send. A badly tuned policy costs time to first token, not behaviour — which is what makes it safe to tune against a real profile.

### Do not register every agent

An agent that runs on a schedule rather than on player attention — a background summariser that fires after every turn, say — should simply not be registered. Evicting it guarantees a restore on its very next turn, which is strictly worse than leaving it resident. Prefill it once during loading instead, with `KvCacheInitialization = Prefill`.

## Step 3 — judge the result against the whole GPU

This is the mistake most worth avoiding. Per-process VRAM tells you **what your server changed**. It does not tell you **whether the game fits**.

The game process, the editor, and the desktop compositor all draw from the same card. On a developer machine with a browser open, the desktop alone can hold 2 GB before your game starts. Reading only the server's figure once led us to conclude that an 8 GB card was out of reach when, on a clean system, it was not.

Measure both, and size your minimum spec against the whole-GPU peak during play.

## What to expect

From the demo, as an order of magnitude rather than a promise:

| Change                                           | Server VRAM |
| ------------------------------------------------ | ----------- |
| Nine contexts at the 8192 default                | 11.7 GB     |
| After sizing (2048 classifiers, 4096 generators) | 8.5 GB      |
| After eviction, hot set of 2                     | 6.8 GB      |

Sizing was the larger win and cost nothing in feel. Eviction added a second win of similar magnitude and a 1–3 s cost on the first line to a newly selected agent.

Note also what did **not** change: a character switch cost ~160 ms before eviction, because every context was already resident. Eviction does not make switching faster — it makes switching cost something, in exchange for memory. If your card has headroom, sizing alone may be the whole answer.

## Related

- [Manage an agent's KV cache](https://docs.tryllengine.com/v0.8/how-to/manage-agent-kv-cache.html) — the underlying prefill/evict/status operations
- [Save and load agent state](https://docs.tryllengine.com/v0.8/how-to/save-and-load-agent-state.html) — pause policy scheduling around export/import so a debounce cannot collide with a snapshot
- [Estimate memory footprint](https://docs.tryllengine.com/v0.8/how-to/estimate-memory-footprint.html) — working out cost before shipping
- [Inspect server memory](https://docs.tryllengine.com/v0.8/how-to/inspect-server-memory.html) — live KV reserved vs context-used after you size or evict
- [Build an immersion guard](https://docs.tryllengine.com/v0.8/how-to/build-an-immersion-guard.html) — adds two classifier contexts per agent, so read this page first
