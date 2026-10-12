# Drive Prompts and Retrieval from Game State

Inject live game state — player level, mood, quest progress — into a prompt template and a `Retrieve` filter, and have both pick up changes on the very next turn with no `ChangeParams` round-trip and no filter recompile.

Prerequisites

- A graph with at least one `Generate`, `GenerateAndSpeak`, or `Transform` node (for the template) and, optionally, a `Retrieve` or `ClassifyIntent` node (for the filter).
- Read [Agent Variables](https://docs.tryllengine.com/v0.8/concepts/agent-variables.html) first for the full model — this page is a task-oriented walkthrough, not the reference.

______________________________________________________________________

## 1. Declare the variables at `CreateAgent`

Declare every variable the graph will ever reference, with its initial value. This locks each name to a type for the agent's lifetime.

In the Inspector, on the `TryllWorkflowAsset`: add to **Variables** — `level` (Int, `1`), `mood` (String, `"neutral"`), `quests_reached` (StringSet, empty).

In the workflow asset's **Variables** array: `level` (Int, `1`), `mood` (String, `"neutral"`), `quests_reached` (StringSet, empty).

```
std::vector<Tryll::Client::AgentVariableDecl> variables = {
    {"level", std::int64_t{1}},
    {"mood",  std::string{"neutral"}},
    {"quests_reached", std::vector<std::string>{}},
};
// variables is the last CreateAgent parameter (there is no agent-name arg).
auto agent = client.CreateAgent(graph, /*enableDiagnostics=*/false,
                                std::nullopt, /*maintainDialogueHistory=*/true, variables);
```

```
agent = client.create_agent(graph, variables={
    "level": 1,
    "mood": "neutral",
    "quests_reached": [],
})
```

______________________________________________________________________

## 2. Reference them in a template

On a `Generate` (or `GenerateAndSpeak`/`Transform`) node's `template` field — **not** `system_prompt`, which is inserted verbatim and never Mustache-rendered:

```
The player is level {{var.level}} and seems {{var.mood}}.
They have completed: {{var.quests_reached}}.
```

A `set<string>` variable renders as its elements **sorted and comma-joined** (e.g. `lost_amulet, silver_key`) — it is a single string value, not an iterable, so Mustache section syntax (`{{#var.quests_reached}}…{{/var.quests_reached}}`) does not enumerate it. Every reference uses the `var.` prefix.

## 3. Reference them in a Retrieve filter

Gate which lore records come back based on the same variables — see [Retrieve filter grammar](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve-filter-grammar.html) for the full operand grammar:

```
{ "op": "and", "args": [
    { "op": "ge", "lhs": { "var": "level" }, "rhs": { "knowledge": "min_level" } },
    { "op": "in", "needle": { "knowledge": "quest_reached" },
                  "haystack": { "var": "quests_reached" } }
] }
```

## 4. Mutate them from gameplay code

Call the typed setters whenever game state changes. Writes update the local mirror immediately; the actual wire update is batched and flushed automatically right before the next send (`SendMessage`; `SendText` in C++; `send_message` in Python), `Resume`, or `ChangeParams` call — you don't need to flush manually.

```
var vars = agentComp.Agent.Variables;
vars.SetInt("level", playerLevel);
vars.SetString("mood", currentMood);
vars.AddToSet("quests_reached", questId);
agentComp.SendMessage(userText); // flush happens automatically before send
```

```
FTryllAgentVariables& Vars = Agent->GetVariables();
Vars.SetInt(TEXT("level"), PlayerLevel);
Vars.SetString(TEXT("mood"), CurrentMood);
Vars.AddToSet(TEXT("quests_reached"), QuestId);
Agent->SendMessage(UserText); // flush happens automatically before send
```

```
auto& vars = agent.Variables();
vars.SetInt("level", playerLevel);
vars.SetString("mood", currentMood);
vars.AddToSet("quests_reached", questId);
agent.SendText(userText); // flush happens automatically before send
```

```
agent.variables.set("level", player_level)
agent.variables.set("mood", current_mood)
agent.variables.add_to_set("quests_reached", quest_id)
agent.send_message(user_text)  # flush happens automatically before send
```

______________________________________________________________________

## Related

- [Agent Variables](https://docs.tryllengine.com/v0.8/concepts/agent-variables.html)
- [Retrieve filter grammar](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve-filter-grammar.html)
- [Use Mustache templates](https://docs.tryllengine.com/v0.8/how-to/use-mustache-templates.html)
- [Change Agent Parameters at Runtime](https://docs.tryllengine.com/v0.8/how-to/change-agent-parameters.html)
