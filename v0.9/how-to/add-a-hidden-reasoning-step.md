# Add a Hidden Reasoning or Draft Step

Sometimes a node should produce text the *conversation* never remembers — an internal draft, a plan, a scratch calculation, or a "thinking out loud" pass that precedes the real answer. Slots make this a two-knob decision that is independent of what the node actually generates:

- **`send`** — does the text go out on the wire to the client?
- **`history_role`** — is the text replayed as an assistant message on later turns?

Setting `history_role = None` is what makes a step "hidden" from the conversation: its output is still written to a slot and is fully usable *this turn* (via `input` or `{{slot.<name>}}`), but it never becomes part of the remembered transcript. `send` then independently decides whether the user sees it happening.

Prerequisites

- A working `Generate` agent — see [Build a chat agent with a graph](https://docs.tryllengine.com/v0.9/how-to/build-chat-agent-with-graph.html).
- Read [Slots and inter-node value passing](https://docs.tryllengine.com/v0.9/concepts/slots-and-inter-node-value-passing.html) for the `send` / `history_role` / `input` model this recipe applies.

## Two variants, one pattern

Both variants are a `think → answer` chain: an upstream node reasons or drafts into its slot, and a downstream `Generate` reads that slot and writes the reply the user actually keeps.

| Variant              | `send`     | `history_role` | The user…                                       |
| -------------------- | ---------- | -------------- | ----------------------------------------------- |
| **Hidden draft**     | `None`     | `None`         | never sees the step; it is pure scratch         |
| **Visible thinking** | `Streamed` | `None`         | sees it stream live, but it is never remembered |

The only difference is `send`. `history_role = None` is identical in both: the step is scratch as far as the *transcript* is concerned.

```
flowchart LR
    think["Generate<br>think<br>(history_role=None)"] -->|input=think| answer["Generate<br>answer<br>(history_role=Assistant)"]
```

## Variant A — a hidden draft

The `think` node reasons privately; `answer` reads its slot and produces the reply. Nothing about the `think` step reaches the client.

```
from tryll_client.graph import GraphDescription
from tryll_client._generated.node_params import (
    GenerateParams, SendAnswer, HistoryRole, SamplingOverrides,
)

think = GenerateParams(
    system_prompt=(
        "Think step by step about how to answer the user's latest "
        "message. Write a short plan. Do NOT answer the user yet."
    ),
    send=SendAnswer.None_,          # never streamed to the client
    history_role=HistoryRole.None_, # never replayed on later turns
    sampling=SamplingOverrides(temperature=0.0),
    default_exit="answer",
)

answer = GenerateParams(
    system_prompt="You are a helpful assistant.",
    # Render the private plan above the user turn (default placement
    # BeforeUserAsSystem):
    template="Your plan:\n{{slot.think}}",
    # send / history_role keep their defaults (Streamed / Assistant).
    default_exit="",                # END
)

graph = (
    GraphDescription()
    .add_node("think", think)
    .add_node("answer", answer)
    .set_start_node("think")
    .set_default_model_name("My Local Model")
)
```

```
using namespace Tryll::NodeParams;

GenerateParamsT think;
think.system_prompt =
    "Think step by step about how to answer the user's latest "
    "message. Write a short plan. Do NOT answer the user yet.";
think.send         = Tryll::SendAnswer_None;    // never streamed
think.history_role = Tryll::HistoryRole_None;   // never replayed
think.default_exit = "answer";

GenerateParamsT answer;
answer.system_prompt = "You are a helpful assistant.";
answer.template_     = "Your plan:\n{{slot.think}}";
// send / history_role keep their defaults (Streamed / Assistant).

Tryll::Client::GraphDescription graph;
graph.AddGenerate("think",  std::move(think))
     .AddGenerate("answer", std::move(answer))
     .SetStartNode("think")
     .SetDefaultModelName("My Local Model");
```

Because `think.send = None`, the client receives `AnswerText` frames only from `answer`. Because `think.history_role = None`, next turn's projected prompt contains the previous answer but no trace of the plan.

## Variant B — visible thinking

Flip `send` to `Streamed` and the reasoning streams to the client live — useful for a "thinking…" panel — while `history_role = None` still keeps it out of the remembered conversation:

```
think = GenerateParams(
    system_prompt="Reason out loud about the user's message.",
    send=SendAnswer.Streamed,       # streamed live to the client
    history_role=HistoryRole.None_, # still never remembered
    default_exit="answer",
)
```

```
GenerateParamsT think;
think.system_prompt = "Reason out loud about the user's message.";
think.send          = Tryll::SendAnswer_Streamed;  // streamed live
think.history_role  = Tryll::HistoryRole_None;      // never remembered
think.default_exit  = "answer";
```

Now the client receives streamed frames from **both** `think` and `answer`. Every `AnswerText` frame carries the producing node's name, so the UI can route the two streams to different places — a dim "thinking" area versus the main reply bubble. See [Send multiple answers in one turn](https://docs.tryllengine.com/v0.9/how-to/send-multiple-answers-in-a-turn.html) for the client-side routing.

## Why the slot is still usable

Both variants set `history_role = None`, yet `answer` reads the step's output via `{{slot.think}}`. That works because **same-turn slot reads are unconditional** — `send` and `history_role` govern the wire and the *remembered* transcript, never whether a downstream node in the same turn can see the slot (see [Slots and inter-node value passing](https://docs.tryllengine.com/v0.9/concepts/slots-and-inter-node-value-passing.html#same-turn-effects)). `history_role = None` only means "don't carry this into future turns."

Draft → refine gets the draft for free

If you set the upstream step to `history_role = Assistant` instead of `None`, its output *also* replays as an assistant message into the downstream node's prompt within the same turn — so a refine node sees the draft both as `{{slot.<name>}}` and as a natural preceding assistant turn. Use `None` when the step is pure scratch; use `Assistant` when you want the downstream model to treat it as a real prior turn.

## See also

- [Slots and inter-node value passing](https://docs.tryllengine.com/v0.9/concepts/slots-and-inter-node-value-passing.html)
- [Send multiple answers in one turn](https://docs.tryllengine.com/v0.9/how-to/send-multiple-answers-in-a-turn.html)
- [Query rewriting for RAG](https://docs.tryllengine.com/v0.9/how-to/query-rewriting-for-rag.html) — the same hidden-step pattern applied to retrieval
- [Transform node](https://docs.tryllengine.com/v0.9/reference/nodes/transform.html) — a model-free hidden step
