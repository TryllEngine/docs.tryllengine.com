# Send Multiple Answers in One Turn

A single turn can have more than one node that sends text to the client — two characters speaking, a "thinking" channel alongside the real reply, or a narrator plus dialogue. Every `AnswerText` frame carries the **name of the node that produced it**, so your UI can tell the streams apart and route each one to the right place.

Prerequisites

- You can already stream a single answer — see [Stream answers to a UI](https://docs.tryllengine.com/v0.8/how-to/stream-answers-to-ui.html).
- Read [Slots and inter-node value passing](https://docs.tryllengine.com/v0.8/concepts/slots-and-inter-node-value-passing.html) for the `send` model this recipe relies on.

## Step 1 — author two sending nodes

Any node with `send ∈ {Whole, Streamed}` contributes to the wire. Put two of them on the same turn's path and give them meaningful names — the name is the attribution key the client sees. Here two `Generate` nodes each speak as a different character:

```
from tryll_client.graph import GraphDescription
from tryll_client._generated.node_params import GenerateParams, SendAnswer

npc_a = GenerateParams(
    system_prompt="You are Ada, a curt engineer. Reply in one sentence.",
    send=SendAnswer.Streamed,   # streams to the client, attributed as "npc_a"
    default_exit="npc_b",
)

npc_b = GenerateParams(
    system_prompt="You are Bram, a cheerful bard. Reply in one sentence.",
    send=SendAnswer.Streamed,   # streams to the client, attributed as "npc_b"
    default_exit="",            # END
)

graph = (
    GraphDescription()
    .add_node("npc_a", npc_a)
    .add_node("npc_b", npc_b)
    .set_start_node("npc_a")
    .set_default_model_name("My Local Model")
)
```

```
using namespace Tryll::NodeParams;

GenerateParamsT npcA;
npcA.system_prompt = "You are Ada, a curt engineer. Reply in one sentence.";
npcA.send          = Tryll::SendAnswer_Streamed;
npcA.default_exit  = "npc_b";

GenerateParamsT npcB;
npcB.system_prompt = "You are Bram, a cheerful bard. Reply in one sentence.";
npcB.send          = Tryll::SendAnswer_Streamed;
// npcB.default_exit stays empty = END

GraphDescription graph;
graph.AddGenerate("npc_a", std::move(npcA))
     .AddGenerate("npc_b", std::move(npcB))
     .SetStartNode("npc_a")
     .SetDefaultModelName("My Local Model");
```

The node **name** (`npc_a`, `npc_b`) is what arrives on each frame. If you set `output_name`, that does *not* change attribution — `node_name` on the wire is always the producing node's name, not its slot name.

## Step 2 — route each stream by node name

Every answer callback receives the node name as its first argument. Switch on it to send each stream to the right widget.

`OnAnswerText` fires on the Unity **main thread** with `(nodeName, text, isDelta, isFinal)`:

```
var agentComp = GetComponent<TryllAgentComponent>();

agentComp.OnAnswerText.AddListener((nodeName, text, isDelta, isFinal) =>
{
    switch (nodeName)
    {
        case "npc_a": adaBubble.AppendText(text);  break;
        case "npc_b": bramBubble.AppendText(text); break;
    }
});
```

On the `UTryllAgentComponent`, bind **On Answer Text (NodeName: FString, Text: FString, bIsFinal: bool)** and branch on `NodeName`:

- `NodeName == "npc_a"` → append `Text` to Ada's `UTextBlock`.
- `NodeName == "npc_b"` → append `Text` to Bram's `UTextBlock`.

The event fires on the game thread, so you can update UMG widgets directly. (The lower-level `FTryllAgent::SetOnAnswerText` callback also exposes an `bIsDelta` flag if you need it.)

`SetOnAnswerText` fires on the client library's reader thread with `(nodeName, text, isDelta, isFinal)` — marshal to your UI thread:

```
agent.SetOnAnswerText(
    [&](std::string_view nodeName, std::string_view text,
        bool /*isDelta*/, bool isFinal)
    {
        if (nodeName == "npc_a")      ada.Append(text);
        else if (nodeName == "npc_b") bram.Append(text);

        if (isFinal) { /* this node's stream is done */ }
    });
```

`set_on_answer_text` fires on the reader thread with `(node_name, text, is_delta, is_final)`:

```
def on_text(node_name, text, is_delta, is_final):
    if node_name == "npc_a":
        ada_buffer.append(text)
    elif node_name == "npc_b":
        bram_buffer.append(text)

agent.set_on_answer_text(on_text)
agent.send_message("Introduce yourselves.")
```

## Ordering and `is_final`

Nodes execute one at a time, so the two streams **never interleave**: every `npc_a` frame arrives before the first `npc_b` frame. Within each node's stream the last frame has `is_final = true`, so `node_name` + `is_final` together delimit each answer:

```
npc_a "He" · npc_a "llo" · npc_a "." (is_final) · npc_b "Hi" · npc_b "!" (is_final)
```

`is_final` marks the end of *one node's* stream, not the end of the turn — a later node may still send. Bind `TurnComplete` for "the whole turn is done".

## Voice: the same attribution on audio

`GenerateAndSpeak` and `Speak` fan TTS audio out the same way — the audio callbacks (`OnTtsAudioFormat` / the Unreal audio delegate) carry the same producing-node name, so a multi-voice scene can route each character's audio to its own speaker. See [Add voice output to an agent](https://docs.tryllengine.com/v0.8/how-to/add-voice-to-an-agent.html).

## Common pitfalls

- **Keying UI on node names couples your game code to the graph.** Renaming a node in the editor propagates through the graph's own references, but it cannot reach `nodeName` string literals in your game code — keep the names you switch on stable, or centralise them in one constant.
- **A `send = None` node sends nothing.** It still writes its slot and can feed downstream nodes, but it produces no `AnswerText` and so never appears in these callbacks — see [Add a hidden reasoning or draft step](https://docs.tryllengine.com/v0.8/how-to/add-a-hidden-reasoning-step.html).
- **Don't rely on `is_final` for "turn over".** With multiple senders you get one `is_final` per node; only `TurnComplete` means the turn ended.

## See also

- [Stream answers to a UI](https://docs.tryllengine.com/v0.8/how-to/stream-answers-to-ui.html)
- [Add a hidden reasoning or draft step](https://docs.tryllengine.com/v0.8/how-to/add-a-hidden-reasoning-step.html)
- [Slots and inter-node value passing](https://docs.tryllengine.com/v0.8/concepts/slots-and-inter-node-value-passing.html)
- [Reference: Wire Protocol](https://docs.tryllengine.com/v0.8/reference/wire-protocol.html)
