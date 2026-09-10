# Agent Variables

Agent Variables are a typed, per-agent key→value store that the game writes over the wire and the server reads in three places: Mustache templates (`{{var.<name>}}`), the Retrieve/ClassifyIntent [filter grammar](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve-filter-grammar.html) (`{"var": "<name>"}`), and — when opted in — **output substitution** of `__NAME__` markers in streamed `Generate` / `GenerateAndSpeak` text. They exist to inject **external game state** — player level, mood, inventory, quest progress — into a prompt, a retrieval filter, or a spoken line without a [`ChangeParam`](https://docs.tryllengine.com/v0.8/how-to/change-agent-parameters.html) round-trip for every change.

**Canonical example.** The player levels up → the game calls `agent.Variables().SetInt("level", 13)` → the *next* turn, the instruction template `"The player is level {{var.level}}."` renders the new value, and a Retrieve filter `var.level >= knowledge.min_level` admits new lore records. No template re-send, no filter re-send, no recompile.

See [Slots vs. Variables](https://docs.tryllengine.com/v0.8/concepts/slots-and-inter-node-value-passing.html#slots-vs-variables) for how this differs from the (superficially similar) `{{slot.<name>}}` mechanism.

______________________________________________________________________

## Declaring variables

Variables are declared **completely** at `CreateAgent` time — the full set of names, types, and initial values the agent will ever have. This declaration locks each variable's type for the agent's whole lifetime; a later write that doesn't match the declared type is rejected (`3014 VariableTypeMismatch`), and a write to an undeclared name is rejected (`3013 UnknownVariable`). There is no way to declare a new variable, or change one's type, after `CreateAgent` — recreate the agent if your schema changes.

Five types are supported: `int` (64-bit), `float` (64-bit), `string`, `bool`, and `set<string>`.

Declare on the workflow asset (`TryllWorkflowAsset.Variables`):

```
// In the Inspector: TryllWorkflowAsset → Variables →
//   Name="level", Type=Int, IntValue=1
//   Name="mood",  Type=String, StringValue="neutral"
//   Name="quests_reached", Type=StringSet
```

A `TryllAgentComponent` may override an asset's declared **initial value** (not name, type, or `allow_output_substitution`) per instance via `VariableOverrides` — the Material-Instance pattern: the asset owns the declaration, the component may only override the value. The Inspector lists every declared variable as an optional row (`[override] name [value]`), matching the Sampling optional-field UX. An override whose name/type no longer matches a declared variable is shown as stale and skipped with a warning at `CreateAgent`.

While the component has a live agent (Play Mode or the Editor **Tryll Chat** window), changing an override updates the live `Agent.Variables` mirror immediately; the server sees the new value on the next deferred flush (before `SendMessage` / `Resume` / `ChangeParams`), or immediately if you click **Flush Now** in the Inspector.

Declare on the workflow asset (`UTryllWorkflowAsset::Variables`). A component's `VariableOverrides` follows the same Material-Instance pattern as Unity: the Details panel lists every declared variable as an optional row (`[override] name [value]`). Stale/duplicate serialized entries appear in a warning group and are ignored at `CreateAgent`.

While the component has a live agent (PIE, or an editor-preview agent attached by **Window → Tryll → Tryll Chat**), changing an override updates the local `FTryllAgentVariables` mirror immediately. The wire flush stays deferred until the next `SendMessage` / `Resume` / `ChangeParams`, or run **Flush Now** under *Live Agent — Variables*. Unchecking an override assigns the **workflow declaration default** (not `Reset()`, which would restore the CreateAgent-time merged initial).

```
#include <tryll/TryllClient.h>

std::vector<Tryll::Client::AgentVariableDecl> variables = {
    {"level", std::int64_t{1}},
    {"mood",  std::string{"neutral"}},
    {"quests_reached", std::vector<std::string>{}},
};
// variables is the last CreateAgent parameter (there is no agent-name arg):
//   CreateAgent(graph, enableDiagnostics, timeout, maintainDialogueHistory, variables)
auto agent = client.CreateAgent(graph, /*enableDiagnostics=*/false,
                                std::nullopt, /*maintainDialogueHistory=*/true, variables);
```

```
from tryll_client.variables import VariableDecl

agent = client.create_agent(graph, variables={
    "level": 1,
    "mood": "neutral",
    "quests_reached": [],   # list/set of str -> declared as set<string>
    # Explicit wrapper when you need declaration metadata:
    "price": VariableDecl(18, allow_output_substitution=True),
})
```

The wire type is inferred from each Python value: `bool` → `bool` (checked *before* `int`, since `bool` is an `int` subclass), `int` → `int`, `float` → `float`, `str` → `string`, `set`/`list` of `str` → `set<string>`. Plain values remain valid; wrap with `VariableDecl` only when you need `allow_output_substitution`.

______________________________________________________________________

## Output substitution (`__NAME__`)

Opt-in replacement of `__<variable-name>__` markers in streamed LLM output. Matching is ASCII case-insensitive. Two switches must both be on:

1. **Declaration:** `allow_output_substitution = true` on the variable (immutable after `CreateAgent`; default `false`). Supported for `int` / `float` / `string` / `bool` only — never `set<string>`. Enabled string values are capped at **2048 UTF-8 bytes** (initial value and every later assignment); non-enabled strings keep the usual 64 KiB limit. Marker-safe names: no `__` substring, must not end with `_`, and no ASCII case-fold collision with another enabled name.
1. **Node:** `substitute_agent_variables = true` on `Generate` or `GenerateAndSpeak` (mutable; default `false`).

Transformed text is authoritative for the output slot, wire answer, dialogue history, and TTS input. Unknown markers and incomplete markers at end-of-stream are left as-is; inserted values are not rescanned. See [Substitute Agent Variables in LLM Output](https://docs.tryllengine.com/v0.8/how-to/substitute-agent-variables-in-output.html) for a walkthrough. This is distinct from input-side `{{var.price}}` templating.

______________________________________________________________________

## Reading and writing

Every client exposes a dedicated variables object reachable from the agent, with one typed setter/getter pair per type plus `AddToSet`/`RemoveFromSet` (idempotent — a no-op success if the element is already present/absent) and `Reset` (restores the `CreateAgent`-time initial value). Reads are **synchronous and local**: each client keeps a mirror that updates immediately on write, so gameplay code never waits on a round-trip to read back what it just set.

```
var vars = agentComp.Agent.Variables; // TryllAgent.Variables property

var err = vars.SetInt("level", 13);
if (!err.IsOk) Debug.LogError($"{err.Code}: {err.Message}");

vars.AddToSet("quests_reached", "lost_amulet");
long level = vars.GetInt("level");
```

```
FTryllAgentVariables& Vars = Agent->GetVariables();

FTryllError Err = Vars.SetInt(TEXT("level"), 13);
if (!Err.IsOk()) UE_LOG(LogTemp, Warning, TEXT("%d: %s"), Err.Code, *Err.Message);

Vars.AddToSet(TEXT("quests_reached"), TEXT("lost_amulet"));
int64 Level = Vars.GetInt(TEXT("level"));
```

Blueprint: `Get Variables` on `TryllAgentComponent`, then `Set Int Variable` / `Set Float Variable` / `Set String Variable` / `Set Bool Variable` / `Set String Set Variable` / `Add To Set Variable` / `Remove From Set Variable` / `Reset Variable`, and the matching `Get*Variable` getters.

```
auto& vars = agent.Variables();
vars.SetInt("level", 13);
vars.AddToSet("quests_reached", "lost_amulet");
std::int64_t level = vars.GetInt("level").value_or(0);
```

```
agent.variables.set("level", 13)          # type inferred from the value
agent.variables.add_to_set("quests_reached", "lost_amulet")
level = agent.variables["level"]          # mapping-style read
```

The Python surface is `set`, `add_to_set`, `remove_from_set`, `reset`, `flush_if_dirty`, and `sync_from_server` (there are no per-type `set_int`/`set_float` methods — `set` infers the type from the value).

An unknown name or a type mismatch is rejected **locally** — synchronously, with no wire round-trip — using the same error codes the server would return (`3013`/`3014`), so a typo in a variable name fails fast in the same place you wrote the bug.

______________________________________________________________________

## Deferred flush — when writes actually reach the server

A write updates the local mirror immediately, but the wire message isn't sent right away. Instead, every client batches pending writes and flushes them as one `UpdateAgentVariablesRequest` immediately before the agent's next `SendMessage` / `Resume` / `ChangeParams` call — never before `Cancel`. This means you can call `SetInt`/`AddToSet`/etc. as often as you like from gameplay code (health changes, mood ticks, quest updates) without worrying about flooding the connection: only one batched frame goes out per turn, right before the turn that needs to see the new values. TCP's ordering guarantee means the flush is always applied *before* the triggering call, so the same turn that picks up the new variable values.

If you need the server's authoritative confirmation before proceeding (rare — most gameplay code doesn't): in Python `flush_if_dirty()` is synchronous and raises on rejection; in Unity/Unreal the explicit flush takes a completion callback. The C++ `Flush()` is fire-and-forget `void` — it returns nothing to await; a rejection surfaces via the agent's `SetOnError` callback (or call `SyncFromServer(...)` to read back the server's values).

______________________________________________________________________

## Busy rule

`UpdateAgentVariablesRequest` is applied **atomically** and admission mirrors [`ChangeAgentParam`](https://docs.tryllengine.com/v0.8/how-to/change-agent-parameters.html): allowed while the agent is idle or [paused](https://docs.tryllengine.com/v0.8/how-to/pause-and-resume-turns.html), rejected with `3004 AgentBusy` while a turn is actively running between pauses. Every update in a batch is validated before any is applied, so a rejected batch never leaves the store partially mutated — the error message names the first invalid update's index.

______________________________________________________________________

## Error codes

| Code                      | Name                   | Cause                                                                                                                                                                                                                                                                                                                                  |
| ------------------------- | ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `3013`                    | `UnknownVariable`      | An `UpdateAgentVariables` **write** targets a name not in `CreateAgentRequest.variables`. (An undeclared name in a *filter* or *template* is caught earlier, at graph compilation — see the `filter.unknown_variable` row and `3003 GraphCompilationFailed`.)                                                                          |
| `3014`                    | `VariableTypeMismatch` | `Assign`/`AddToSet`/`RemoveFromSet` value's type doesn't match the variable's declared type.                                                                                                                                                                                                                                           |
| `3015`                    | `InvalidVariableValue` | The `CreateAgent` declaration itself is invalid (bad name grammar, duplicate name, name/string/set over the length limit, `allow_output_substitution` on a set, enabled-string over 2048 bytes, or marker-unsafe / case-colliding enabled name). The same code rejects Assign/Reset that would push an enabled string past 2048 bytes. |
| `3004`                    | `AgentBusy`            | `UpdateAgentVariablesRequest` arrived while a turn is actively running (not idle, not paused).                                                                                                                                                                                                                                         |
| `filter.unknown_variable` | —                      | A Retrieve/ClassifyIntent filter's `{"var": "<name>"}` operand names an undeclared variable; fails graph compilation (`3003 GraphCompilationFailed`).                                                                                                                                                                                  |

______________________________________________________________________

## Related

- [Slots and inter-node value passing](https://docs.tryllengine.com/v0.8/concepts/slots-and-inter-node-value-passing.html)
- [Retrieve filter grammar](https://docs.tryllengine.com/v0.8/reference/nodes/retrieve-filter-grammar.html)
- [Use Mustache templates](https://docs.tryllengine.com/v0.8/how-to/use-mustache-templates.html)
- [Change Agent Parameters at Runtime](https://docs.tryllengine.com/v0.8/how-to/change-agent-parameters.html)
- [How-to: drive prompts and retrieval from game state](https://docs.tryllengine.com/v0.8/how-to/drive-prompts-and-retrieval-from-game-state.html)
- [How-to: substitute agent variables in LLM output](https://docs.tryllengine.com/v0.8/how-to/substitute-agent-variables-in-output.html)
- [How-to: keep merchant prices deterministic](https://docs.tryllengine.com/v0.8/how-to/keep-merchant-prices-deterministic.html)
