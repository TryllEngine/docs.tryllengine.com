# Build a Chat Agent with a Graph

Create an agent whose graph is a simple `RegexGuardrail → Generate` chain, send a message, and get a streamed reply.

Prerequisites

- A connected session with `CreateSession` already called — see [Connect and Manage a Session](https://docs.tryllengine.com/v0.8/how-to/connect-and-manage-session.html).
- A model listed in `models.json` that is either already `Loaded` or at least `Local`. If not, see [Use Your Own Local Model](https://docs.tryllengine.com/v0.8/how-to/use-your-own-local-model.html).

## Steps

1. In the Project window: right-click → **Create → Tryll → Workflow Asset**. Name it `ChatWorkflow`.
1. In the Inspector, expand **Graph** and add two nodes:
   - `guard` — **Params** type `RegexGuardrail`. Leave **Triggered Exit** empty (END) to drop jailbreaks silently, and set **Not Triggered Exit** = `answer`.
   - `answer` — **Params** type `Generate`. **Default Exit** stays empty (END).

Set **Start Node** = `guard` and **Default Model Name** = your model. 3. Attach a `TryllAgentComponent` to a GameObject and assign `ChatWorkflow` to its **Workflow Asset** field. 4. Wire events and send:

```
var agentComp = GetComponent<TryllAgentComponent>();

agentComp.OnAnswerText.AddListener(
    (nodeName, text, isDelta, isFinal) => chatText.text += text);
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
namespace NP = Tryll::NodeParams;

NP::RegexGuardrailParamsT guardParams;
// guardParams.triggered_exit defaults to "" (END) — drop jailbreaks silently.
guardParams.not_triggered_exit = "answer";

TC::GraphDescription graph;
graph.AddRegexGuardrail("guard", guardParams)
     .AddGenerate("answer", NP::GenerateParamsT{})
     .SetStartNode("guard")
     .SetDefaultModelName("My Local Model");

auto agent = client.CreateAgent(graph);

// Register persistent callbacks, then send — fire-and-forget.
agent.SetOnAnswerText(
    [](std::string_view /*nodeName*/, std::string_view text, bool, bool)
    { std::cout << text << std::flush; });
agent.SetOnTurnComplete([](::Tryll::TurnStatus, std::string_view, std::int32_t)
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

Server-side, with `log_level` set to `"trace"` in [`server-config.json`](https://docs.tryllengine.com/v0.8/reference/server-configuration.html), each node logs a start and an end line that names the exit it took. One turn through this graph includes:

```
[trace] [WORKFLOW] start  entry=ExecuteOnMessage
[trace] [NODE]     start  node=guard
[trace] [NODE]     end    node=guard  exit=not_triggered  duration_ms=0.2
[trace] [NODE]     start  node=answer
[trace] [NODE]     end    node=answer  exit=default  duration_ms=812.4
[trace] [WORKFLOW] end    status=success  total_ms=813.0
```

Client-side, Python returns the full reply string from `send_message`, while Unity, C++, and Unreal see each streamed chunk arrive in the `OnAnswerText` event / `SetOnAnswerText` callback / `OnAnswerText` delegate, followed by a `TurnComplete` with status `Success`.

## Common pitfalls

- **Unknown or unacquired model.** A graph whose model name is wrong, missing from `models.json`, or not yet on disk fails `CreateAgent` with error [3003 `GraphCompilationFailed`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#3xxx-agent). Calling `LoadModel` with a name that is not in the catalog fails with [6005 `ModelResolutionFailed`](https://docs.tryllengine.com/v0.8/reference/error-codes.html#6xxx-model-download). Double-check the name with `client.list_models()`.
- **"Graph validation failed"** (error [3003](https://docs.tryllengine.com/v0.8/reference/error-codes.html#3xxx-agent)) — check for a bad start node name or duplicate node names in the graph. The same code covers an exit field that names a node not in the graph — fix the string, or leave it empty to route to END.
- **Silent nothing after sending.** Check that the start node is not an always-`END` branch (e.g., guardrail that matches everything). The server log will show which route was taken.

## Related

- [Reference: Workflow Nodes](https://docs.tryllengine.com/v0.8/reference/nodes/index.html)
- [Reference: Agent Parameters](https://docs.tryllengine.com/v0.8/reference/agent-parameters.html)
- [Concept: Workflows, Graphs, and Nodes](https://docs.tryllengine.com/v0.8/concepts/workflows-graphs-nodes.html)
- [How-to: Edit Workflows in the Graph Editor (Unity)](https://docs.tryllengine.com/v0.8/how-to/edit-workflows-in-graph-editor.html)
- [How-to: Use Canned Responses and Guardrails](https://docs.tryllengine.com/v0.8/how-to/use-canned-responses-and-guardrails.html)
- [How-to: Stream Answers to a UI](https://docs.tryllengine.com/v0.8/how-to/stream-answers-to-ui.html)
