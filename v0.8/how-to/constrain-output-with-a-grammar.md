# Constrain Output with a Grammar

Force a [`Generate`](https://docs.tryllengine.com/v0.8/reference/nodes/generate.html) node to emit exactly one of a fixed set of machine-readable answers — a command, a choice, a tiny JSON object — by attaching a [GBNF](https://github.com/ggml-org/llama.cpp/blob/master/grammars/README.md) grammar. Every reply is then guaranteed to match the grammar; there are no parse failures to handle.

Prerequisites

- A connected session, with `CreateSession` already called.
- A language model on disk. For command parsing, prefer a model with thinking **off** (per-variant `disable_thinking`) — see [Model Management](https://docs.tryllengine.com/v0.8/reference/model-management.html).
- Read [Constrained Output (GBNF)](https://docs.tryllengine.com/v0.8/concepts/constrained-output.html) for what a grammar does and does **not** guarantee (syntax, never semantics).

## Step 1 — write the grammar

A grammar is a set of rules. It **must** have a rule named `root`, and it must not be left-recursive. Keep the value sets small and use words that tokenise cleanly:

```
root   ::= action " " object
action ::= "grab" | "pull" | "cut" | "activate"
object ::= "wire" | "enemy" | "lever" | "door"
```

Every output is one of the 4×4 = 16 strings. To let the model signal a missing/unrecognised slot, add a sentinel as an alternative in each slot (e.g. `object ::= "wire" | … | "unknown"`) — but note the model only *uses* the sentinel if you tell it to (Step 2) and it follows the instruction (verify in Step 3).

## Step 2 — put it on a `Generate` node

Set `grammar` alongside a `system_prompt` that **spells out the allowed words and the output order**. The grammar enforces the shape; the prompt drives which words get chosen.

```
from tryll_client.graph import GraphDescription, GenerateParams, SamplingOverrides

GRAMMAR = (
    'root   ::= action " " object\n'
    'action ::= "grab" | "pull" | "cut" | "activate"\n'
    'object ::= "wire" | "enemy" | "lever" | "door"'
)
PROMPT = (
    "You control a game character. Reply with the single best command as "
    "'<action> <object>', choosing one action from {grab, pull, cut, "
    "activate} and one object from {wire, enemy, lever, door}. "
    "Output nothing else."
)

graph = (
    GraphDescription()
    .add_node("command", GenerateParams(
        system_prompt=PROMPT,
        grammar=GRAMMAR,
        sampling=SamplingOverrides(temperature=0.0),  # extraction, not creativity
    ))
    .set_start_node("command")
    .set_default_model_name("My Local Model")
)
agent = client.create_agent(graph)
```

```cpp
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

GenerateParamsT gp;
gp.system_prompt = "You control a game character. Reply with the single "
"best command as ' ' ... Output nothing else.";
gp.grammar = R"(root ::= action " " object

action ::= "grab" | "pull" | "cut" | "activate"
object ::= "wire" | "enemy" | "lever" | "door")";
gp.sampling = std::make_unique();
gp.sampling->temperature = 0.0f;
GenerateParamsT commandBaseline = gp; // kept for ChangeParams in step 4

```

GraphDescription graph; graph.AddGenerate("command", std::move(gp)) .SetStartNode("command") .SetDefaultModelName("My Local Model"); auto agent = client.CreateAgent(graph);

```
```

```
using Tryll.Client;

const string grammar =
    "root   ::= action \" \" object\n" +
    "action ::= \"grab\" | \"pull\" | \"cut\" | \"activate\"\n" +
    "object ::= \"wire\" | \"enemy\" | \"lever\" | \"door\"";

var graph = new TryllGraphBuilder()
    .AddGenerate("command", new TryllGenerateParams
    {
        SystemPrompt = "You control a game character. Reply with '<action> <object>' ...",
        Grammar      = grammar,
        Sampling     = new TryllSamplingOverrides { Temperature = 0.0f },
    })
    .SetStartNode("command")
    .SetDefaultModelName("My Local Model")
    .Build();
```

```
#include "Generated/TryllGraphBuilder.Nodes.h"
#include "Generated/TryllNodeParamsFactory.h"

UTryllGenerateParams* P = UTryllNodeParamsFactory::MakeGenerateParams(this);
P->bOverrideSystemPrompt = true;
P->SystemPrompt = TEXT("You control a game character. Reply with '<action> <object>' ...");
P->bOverrideGrammar = true;
P->Grammar = TEXT("root   ::= action \" \" object\n"
                  "action ::= \"grab\" | \"pull\" | \"cut\" | \"activate\"\n"
                  "object ::= \"wire\" | \"enemy\" | \"lever\" | \"door\"");

FTryllGraphDescription Graph = FTryllGraphBuilder()
    .AddNode(TEXT("command"), P)
    .SetStartNode(TEXT("command"))
    .SetDefaultModelName(TEXT("My Local Model"))
    .Build();
```

If the grammar is malformed, `CreateAgent` fails fast with [`GraphCompilationFailed`](https://docs.tryllengine.com/v0.8/reference/error-codes.html) and a message that says the grammar could not be parsed — fix it and recreate.

## Step 3 — verify (and watch for valid-but-wrong)

Send a few messages and confirm every reply is a legal command:

```
for msg in ["cut the wire", "grab the enemy", "open the door", "cat the wiiire"]:
    print(msg, "->", agent.send_message(msg))
# cut the wire   -> cut wire
# grab the enemy -> grab enemy
# open the door  -> pull door      # valid-but-wrong: "open" isn't in the set
# cat the wiiire -> cut wire        # garble recovered
```

Every output matches the grammar. The `open the door` case shows the key caveat: the grammar guarantees a *legal* command, not the *right* one — "open" isn't in the vocabulary, so the model picks the closest legal verb. If that matters, add a gate (Step 5).

## Step 4 — flip between command turn and free chat (optional)

`grammar` is mutable. Clear it to let the same node answer in free-form prose, then set it again for the next command turn — no agent recreation. Python can change the one field by path. C++, Unity, and Unreal send the node's whole params through `ChangeParams`, so start from a copy of the authored params. See [Change agent parameters](https://docs.tryllengine.com/v0.8/how-to/change-agent-parameters.html).

```
agent.change_param("command", "grammar", "")      # free chat this turn
agent.send_message("Nice work. What should we grab next?")

agent.change_param("command", "grammar", GRAMMAR) # back to command mode
```

```
GenerateParamsT freeChat = commandBaseline;
freeChat.grammar = "";
agent.ChangeParams("command", std::move(freeChat));  // free chat
// ... later ...
agent.ChangeParams("command", commandBaseline);      // command mode
```

```
var freeChat = (TryllGenerateParams)agent.GetNodeParamsBaseline("command");
freeChat.Grammar = "";
agent.ChangeParams("command", freeChat);             // free chat

var command = (TryllGenerateParams)agent.GetNodeParamsBaseline("command");
command.Grammar = grammar;
agent.ChangeParams("command", command);              // command mode
```

```
auto* Base = Cast<UTryllGenerateParams>(AgentComp->GetNodeParamsBaseline(TEXT("command")));

auto* FreeChat = Cast<UTryllGenerateParams>(UTryllNodeParamsFactory::CloneParams(Base, this));
FreeChat->bOverrideGrammar = true;
FreeChat->Grammar = TEXT("");
AgentComp->ChangeParams(TEXT("command"), FreeChat);  // free chat

auto* Command = Cast<UTryllGenerateParams>(UTryllNodeParamsFactory::CloneParams(Base, this));
Command->bOverrideGrammar = true;
Command->Grammar = Grammar;
AgentComp->ChangeParams(TEXT("command"), Command);   // command mode
```

A malformed grammar passed here is rejected with [`InvalidParamValue` (3007)](https://docs.tryllengine.com/v0.8/reference/error-codes.html) and the previous grammar is kept.

## Step 5 — gate low-confidence input (optional)

Grammar output carries no confidence, so a garbled utterance becomes a confident wrong command. To *reject* rather than guess, put a [`Classify Intent (LLM)`](https://docs.tryllengine.com/v0.8/reference/nodes/classify-intent-llm.html) node in front: it returns the verb **with a calibrated probability** and threshold/margin gates, so an out-of-scope utterance routes `not_found` instead of reaching the grammar node. The grammar node then only handles turns already known to be commands. See [Constrained Output → grammar vs tool calling vs intent classification](https://docs.tryllengine.com/v0.8/concepts/constrained-output.html#grammar-vs-tool-calling-vs-intent-classification).

## Common pitfalls

- **Valid ≠ correct.** The model will emit a legal-but-wrong value on input the vocabulary can't express. Only your checks (or a Step-5 gate) catch it.
- **The sentinel is opt-in *and* model-dependent.** Adding `"unknown"` to the grammar lets the model say "don't know"; whether it actually does is instruction-following that varies by model. Spell the rule out in the prompt and measure it.
- **Reasoning models need thinking off.** A grammar that forbids `<think>` tokens breaks generation. Use a non-thinking variant.
- **Runaway grammars hit `max_tokens`.** A grammar with no stopping path (e.g. `root ::= "a" root`) generates until the cap. Give constrained nodes a small `max_tokens`.

## Related

- [Concept: Constrained Output (GBNF)](https://docs.tryllengine.com/v0.8/concepts/constrained-output.html)
- [Reference: Generate node](https://docs.tryllengine.com/v0.8/reference/nodes/generate.html#constrained-output-gbnf-grammar)
- [How-to: Change agent parameters at runtime](https://docs.tryllengine.com/v0.8/how-to/change-agent-parameters.html)
- [Reference: Classify Intent (LLM) node](https://docs.tryllengine.com/v0.8/reference/nodes/classify-intent-llm.html)
- [How-to: Define and handle tool calls](https://docs.tryllengine.com/v0.8/how-to/define-and-handle-tool-calls.html)
