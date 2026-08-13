# Build a Chat Agent with a Graph

Create an agent whose graph is a simple `RegexGuardrail → Generate` chain, send a message, and get a streamed reply.

Prerequisites

- A connected session with `CreateSession` already called — see [Connect and Manage a Session](https://docs.tryllengine.com/how-to/connect-and-manage-session.html).
- A model listed in `models.json` that is either already `Loaded` or at least `Local`. If not, see [Use Your Own Local Model](https://docs.tryllengine.com/how-to/use-your-own-local-model.html).

## Steps

1. In the Project window: right-click → **Create → Tryll → Workflow Asset**. Name it `ChatWorkflow`.
1. In the Inspector, expand **Graph** and add two nodes:
   - `guard` — **Params** type `RegexGuardrail`. Leave **Triggered Exit** empty (END) to drop jailbreaks silently, and set **Not Triggered Exit** = `answer`.
   - `answer` — **Params** type `Generate`. **Default Exit** stays empty (END).

Set **Start Node** = `guard` and **Default Model Name** = your model. 3. Attach a `TryllAgentComponent` to a GameObject and assign `ChatWorkflow` to its **Workflow Asset** field. 4. Wire events and send:

```
var agentComp = GetComponent<TryllAgentComponent>();

agentComp.OnAnswerText.AddListener(
    (text, isDelta, isFinal) => chatText.text += text);
agentComp.OnTurnComplete.AddListener(
    (status, _, _) => typingIndicator.SetActive(false));

// When the agent is created, send the first message.
agentComp.OnAgentCreated.AddListener(() => agentComp.SendMessage("Hello!"));
```

1. In the Content Browser, right-click → **Tryll → Workflow Asset**. Name it `ChatWorkflow` and open it. On the **guard** node, set **Triggered Exit** to `""` (END) and **Not Triggered Exit** to `"answer"`. The **answer** node's **Default Exit** can be left empty (END).
1. Add a `UTryllAgentComponent` to your actor and assign `ChatWorkflow` to its **Workflow Asset** slot.
1. In Blueprint:
   - Bind **On Agent Ready** to a handler that calls **Send Message**.
   - Bind **On Answer Text** to append each chunk to a UI text block.
   - Bind **On Turn Complete** to flip the "typing" indicator off.

```
#include <tryll/TryllClient.h>

namespace TC = Tryll::Client;

TC::RegexGuardrailParams guardParams;
// guardParams.triggered_exit defaults to "" (END) — drop jailbreaks silently.
guardParams.not_triggered_exit = "answer";

TC::GraphDescription graph;
graph.AddNode("guard",  guardParams)
     .AddNode("answer", TC::GenerateParams{})
     .SetStartNode("guard")
     .SetDefaultModelName("My Local Model");

auto agent = client.CreateAgent(graph);

// Register persistent callbacks, then send — fire-and-forget.
agent.SetOnAnswerText([](std::string_view text, bool, bool)
    { std::cout << text << std::flush; });
agent.SetOnTurnComplete([](TC::TurnStatus, std::string_view, std::int32_t)
    { std::cout << "\n"; });
agent.SendText("Hello!");
```

```
from tryll_client.graph import GraphDescription, RegexGuardrailParams, GenerateParams

# 1. Build the graph.
guard_params = RegexGuardrailParams(
    # triggered_exit defaults to "" (END) — drop jailbreaks silently.
    not_triggered_exit = "answer",
)

graph = (
    GraphDescription()
    .add_node("guard",  guard_params)
    .add_node("answer", GenerateParams())
    .set_start_node("guard")
    .set_default_model_name("My Local Model")
)

# 2. Create the agent.
agent = client.create_agent(graph)

# 3. Send a message. send_message blocks and returns the full reply.
reply = agent.send_message("Hello!")
print(reply)
```

## Verify it worked

Server-side, at `info` log level, one turn produces:

```
[info] Agent 7: turn starting
[info] Node guard: not_triggered
[info] Node answer: default
[info] Agent 7: turn complete (Success, tokens=128)
```

Client-side, Python returns the full reply string from `send_message`, while C++ and Unreal see each streamed chunk arrive in the `SendText` callback / `OnAnswerText` delegate, terminated by a `TurnComplete` with status `Success`.

## Common pitfalls

- **"Model not found"** (error [4002](https://docs.tryllengine.com/reference/error-codes.html#4xxx-inference-workflow)) — either the name is wrong, or the model is not in `models.json`. Double-check with `client.list_models()`.
- **"Graph validation failed"** (error [3003](https://docs.tryllengine.com/reference/error-codes.html#3xxx-agent)) — check for a bad start node name or duplicate node names in the graph. If the message says `InvalidExitTarget` ([3008](https://docs.tryllengine.com/reference/error-codes.html#3xxx-agent)), an exit field on one of the nodes names a node that doesn't exist — fix the string or leave it empty for END.
- **Silent nothing after `SendMessage`.** Check that the start node is not an always-`END` branch (e.g., guardrail that matches everything). The server log will show which route was taken.

## Related

- [Reference: Workflow Nodes](https://docs.tryllengine.com/reference/nodes/index.html)
- [Reference: Agent Parameters](https://docs.tryllengine.com/reference/agent-parameters.html)
- [Concept: Workflows, Graphs, and Nodes](https://docs.tryllengine.com/concepts/workflows-graphs-nodes.html)
- [How-to: Edit Workflows in the Graph Editor (Unity)](https://docs.tryllengine.com/how-to/edit-workflows-in-graph-editor.html)
- [How-to: Use Canned Responses and Guardrails](https://docs.tryllengine.com/how-to/use-canned-responses-and-guardrails.html)
- [How-to: Stream Answers to a UI](https://docs.tryllengine.com/how-to/stream-answers-to-ui.html)
