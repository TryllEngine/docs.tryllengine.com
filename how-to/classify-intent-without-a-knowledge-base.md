# Classify Intent Without a Knowledge Base

Route a player's line to a tone-specific reply using [`ClassifyIntentLLM`](https://docs.tryllengine.com/reference/nodes/classify-intent-llm.html) — **no embedding model, no labelled example corpus, no files on disk**. You describe each intent in a sentence, and the language model you are already loading for dialogue reads the answer off its own first-token logprobs.

This is the same gate-guard NPC as [Build an Intent-Driven NPC](https://docs.tryllengine.com/how-to/build-intent-driven-npc.html), rebuilt without the knowledge base — read the two side by side to choose between them.

Prerequisites

- A connected session, created with the `LlamaCpp` engine.
- **One language model** (e.g. `"Llama 3.2 3B Instruct (Q4_K_M)"`). That is the whole requirement — the classifier and the NPC can share it.
- Familiarity with [graphs and exits](https://docs.tryllengine.com/concepts/workflows-graphs-nodes.html) and [slots](https://docs.tryllengine.com/concepts/slots-and-inter-node-value-passing.html).

______________________________________________________________________

## Which classifier should you use?

|                              | [Classify Intent](https://docs.tryllengine.com/reference/nodes/classify-intent.html) (embedding) | Classify Intent (LLM)                  |
| ---------------------------- | ------------------------------------------------------------------------------------------------ | -------------------------------------- |
| **Needs a knowledge base**   | Yes — 5–10 labelled phrases per intent                                                           | **No**                                 |
| **Needs an embedding model** | Yes                                                                                              | No                                     |
| **Defining an intent**       | Add example phrases, re-index                                                                    | Write one sentence                     |
| **Extra inference cost**     | One embedding pass (very fast)                                                                   | One LM prefill (~20 ms on a 1B model)  |
| **Confidence signal**        | Cosine distance                                                                                  | Softmax probability + margin gate      |
| **Practical label ceiling**  | Large — hundreds                                                                                 | ~10–15 before accuracy degrades        |
| **Best for**                 | Many intents, pre-labelled data, tightest latency                                                | Few intents, no corpus, fast iteration |

Reach for the LLM node when you are still *designing* the intents. Writing a sentence and re-running is a much shorter loop than authoring twenty example phrases and rebuilding an index.

______________________________________________________________________

## Overview

```
flowchart LR
    classify["ClassifyIntentLLM\nclassify"]
    pick["IntentToInstruction\npick"]
    compose["Transform\ncompose"]
    respond["Generate\nrespond"]
    classify -- "found" --> pick
    classify -- "not_found" --> compose
    pick -- "default" --> compose
    compose -- "default" --> respond
    respond -- "default" --> END
```

`classify` scores the player's line against four described labels and attaches an `IntentionComponent` on `found`. `pick` maps that label to a tone directive from an inline map and writes it to a slot. `compose` folds the directive above the player's line via Mustache, and `respond` consumes that slot via `input`. The catch-all routes to `not_found`, where `compose` still runs with an empty `{{#instructions}}` block — so `respond` always reads from the same slot and the guard just answers normally.

Downstream of `classify`, this is byte-for-byte the wiring of the embedding-based recipe. Only the classifier node changed.

______________________________________________________________________

## Step 1 — describe the intents

Two parallel comma-separated lists. `intents_ids` is what your graph routes on; `intents_prompt` is what the model actually reads. **Order pairs them positionally** — entry 0 of both is letter `A`, entry 1 is `B`, and so on.

```
INTENT_IDS = "insult,pleasing,joking,other"

INTENT_DESCRIPTIONS = (
    "The traveller insults mocks or belittles the guard,"
    "The traveller is polite deferential or flattering while asking to pass,"
    "The traveller tells a joke pun or riddle,"
    "Anything else - directions questions small talk or an ordinary request"
)
```

No commas inside a description

Both fields split on `,` with no escaping, so a comma inside a description silently becomes an extra label — surfacing later as a confusing count-mismatch error. Use dashes and semicolons.

The catch-all goes **last**, deliberately: these models have a measurable pull toward whichever label is listed first, so the label you most want read honestly should not be sitting at `A`. See [Design an Intent-Classifier Prompt](https://docs.tryllengine.com/how-to/design-an-intent-classifier-prompt.html#order-the-labels-deliberately).

______________________________________________________________________

## Step 2 — write the system prompt

This is the step with no equivalent in the embedding recipe, and the one most likely to catch you out. `system_prompt` is a **Mustache template** with two variables, and there is **no default** — leave it empty and the model is asked to pick a letter without ever being shown what the letters mean.

```
CLASSIFIER_SYSTEM = (
    "You are an intent classifier for a fantasy city gate.\n"
    "A traveller (user) is speaking to Aldric the gate guard (assistant).\n"
    "Pick the single best label for the traveller's LATEST message only.\n\n"
    "Labels:\n{{intents_block}}\n\n"
    "Respond with a single letter (one of {{letters}}). No other text."
)
```

`{{intents_block}}` renders your descriptions as a letter-prefixed list, and `{{letters}}` renders `A, B, C, D`. So the model sees:

```
You are an intent classifier for a fantasy city gate.
A traveller (user) is speaking to Aldric the gate guard (assistant).
Pick the single best label for the traveller's LATEST message only.

Labels:
A. The traveller insults mocks or belittles the guard
B. The traveller is polite deferential or flattering while asking to pass
C. The traveller tells a joke pun or riddle
D. Anything else - directions questions small talk or an ordinary request

Respond with a single letter (one of A, B, C, D). No other text.
```

The player's message is **not** part of this template — the projection adds it as its own user turn, after the system message and any history.

For what to put in the template and what to keep out of it, see [Design an Intent-Classifier Prompt](https://docs.tryllengine.com/how-to/design-an-intent-classifier-prompt.html).

______________________________________________________________________

## Step 3 — build the graph

`not_found_intent="other"` is doing something worth noticing: when the catch-all wins, the node exits **`not_found`** rather than `found`. So the graph has one fall-through path for both "the model chose the catch-all" and "the model was not confident enough", and you only wire tone directives for the three intents that have one.

```
from tryll_client.graph import (
    GraphDescription, ClassifyIntentLLMParams, IntentToInstructionParams,
    TransformParams, GenerateParams, NotifyClient,
)

SYSTEM_PROMPT = (
    "You are roleplaying as Aldric, a grumpy gate guard. "
    "Stay in character at all times. Use short sentences."
)

TEMPLATE = (
    "{{#instructions}}[Tone directive: {{text}}]\n{{/instructions}}"
    "Traveller says: {{user_message}}"
)

graph = (
    GraphDescription()
    .add_node("classify", ClassifyIntentLLMParams(
        intents_ids=INTENT_IDS,
        intents_prompt=INTENT_DESCRIPTIONS,
        system_prompt=CLASSIFIER_SYSTEM,
        history_turns=2,
        threshold=0.5,          # tune this — see step 5
        margin=0.15,
        not_found_intent="other",
        notify_client=NotifyClient.Always,
        found_exit="pick",
        not_found_exit="compose",
    ))
    .add_node("pick", IntentToInstructionParams(
        inline_keys=["insult", "pleasing", "joking"],
        inline_strings=[
            "The traveller has insulted you. Respond curtly and insult them back — short and sharp.",
            "The traveller is being polite. Stay grumpy but show grudging satisfaction. One sentence.",
            'The traveller told a joke. Permit yourself one brief laugh (e.g. "Ha.") then refocus on duty.',
        ],
        output_name="tone",
        default_exit="compose",
    ))
    .add_node("compose", TransformParams(
        template=TEMPLATE,
        default_exit="respond",
    ))
    .add_node("respond", GenerateParams(
        system_prompt=SYSTEM_PROMPT,
        input="compose",
        # default_exit is "" (END) by default.
    ))
    .set_start_node("classify")
    .set_default_model_name("Llama 3.2 3B Instruct (Q4_K_M)")
)

agent = client.create_agent(graph, enable_diagnostics=True)
```

```
using namespace Tryll::Client;
using namespace Tryll::NodeParams;

constexpr const char* kSystemPrompt =
    "You are roleplaying as Aldric, a grumpy gate guard. "
    "Stay in character at all times. Use short sentences.";

constexpr const char* kTemplate =
    "{{#instructions}}[Tone directive: {{text}}]\n{{/instructions}}"
    "Traveller says: {{user_message}}";

ClassifyIntentLLMParamsT classifyParams;
classifyParams.intents_ids      = kIntentIds;       // "insult,pleasing,joking,other"
classifyParams.intents_prompt   = kIntentDescriptions;
classifyParams.system_prompt    = kClassifierSystem;
classifyParams.history_turns    = 2;
classifyParams.threshold        = 0.5f;
classifyParams.margin           = 0.15f;
classifyParams.not_found_intent = "other";
classifyParams.notify_client    = Tryll::NotifyClient_Always;
classifyParams.found_exit       = "pick";
classifyParams.not_found_exit   = "compose";

IntentToInstructionParamsT pickParams;
pickParams.inline_keys    = {"insult", "pleasing", "joking"};
pickParams.inline_strings = {
    "The traveller has insulted you. Respond curtly and insult them back — short and sharp.",
    "The traveller is being polite. Stay grumpy but show grudging satisfaction. One sentence.",
    "The traveller told a joke. Permit yourself one brief laugh then refocus on duty.",
};
pickParams.output_name  = "tone";
pickParams.default_exit = "compose";

TransformParamsT composeParams;
composeParams.template_    = kTemplate;
composeParams.default_exit = "respond";

GenerateParamsT respondParams;
respondParams.system_prompt = kSystemPrompt;
respondParams.input         = "compose";
// respondParams.default_exit is "" (END) by default.

GraphDescription graph;
graph.AddClassifyIntentLLM("classify", std::move(classifyParams))
     .AddIntentToInstruction("pick",   std::move(pickParams))
     .AddTransform("compose",          std::move(composeParams))
     .AddGenerate("respond",           std::move(respondParams))
     .SetStartNode("classify")
     .SetDefaultModelName("Llama 3.2 3B Instruct (Q4_K_M)");

auto agent = client.CreateAgent(graph);
```

```
const string systemPrompt =
    "You are roleplaying as Aldric, a grumpy gate guard. " +
    "Stay in character at all times. Use short sentences.";

const string template =
    "{{#instructions}}[Tone directive: {{text}}]\n{{/instructions}}" +
    "Traveller says: {{user_message}}";

var graph = new TryllGraphBuilder()
    .AddClassifyIntentLLM("classify", new TryllClassifyIntentLLMParams
    {
        IntentsIds     = IntentIds,          // "insult,pleasing,joking,other"
        IntentsPrompt  = IntentDescriptions,
        SystemPrompt   = ClassifierSystem,
        HistoryTurns   = 2,
        Threshold      = 0.5f,
        Margin         = 0.15f,
        NotFoundIntent = "other",
        NotifyClient   = TryllNotifyClient.Always,
        FoundExit      = "pick",
        NotFoundExit   = "compose",
    })
    .AddIntentToInstruction("pick", new TryllIntentToInstructionParams
    {
        InlineKeys    = new[] { "insult", "pleasing", "joking" },
        InlineStrings = new[] {
            "The traveller has insulted you. Respond curtly and insult them back — short and sharp.",
            "The traveller is being polite. Stay grumpy but show grudging satisfaction. One sentence.",
            "The traveller told a joke. Permit yourself one brief laugh then refocus on duty.",
        },
        OutputName  = "tone",
        DefaultExit = "compose",
    })
    .AddTransform("compose", new TryllTransformParams
    {
        Template    = template,
        DefaultExit = "respond",
    })
    .AddGenerate("respond", new TryllGenerateParams
    {
        SystemPrompt = systemPrompt,
        Input        = "compose",
        // DefaultExit is "" (END) by default.
    })
    .SetStartNode("classify")
    .SetDefaultModelName("Llama 3.2 3B Instruct (Q4_K_M)")
    .Build();

var (agent, error) = await TryllClient.Instance.RequestCreateAgentAsync(graph);
agent.SetOnAnswerText((nodeName, text, isDelta, isFinal) => { /* stream text */ });
agent.SetOnTurnComplete((status, debugInfo, tokens) => { /* turn ended */ });
```

```
#include "Generated/TryllGraphBuilder.Nodes.h"
#include "Generated/TryllNodeParamsFactory.h"

UTryllClassifyIntentLLMParams* ClassifyP =
    UTryllNodeParamsFactory::MakeClassifyIntentLLMParams(this);
ClassifyP->bOverrideIntentsIds    = true;
ClassifyP->IntentsIds             = TEXT("insult,pleasing,joking,other");
ClassifyP->bOverrideIntentsPrompt = true;
ClassifyP->IntentsPrompt          = IntentDescriptions;
ClassifyP->bOverrideSystemPrompt  = true;
ClassifyP->SystemPrompt           = ClassifierSystem;
ClassifyP->HistoryTurns           = 2;
ClassifyP->Threshold              = 0.5f;
ClassifyP->Margin                 = 0.15f;
ClassifyP->bOverrideNotFoundIntent = true;
ClassifyP->NotFoundIntent         = TEXT("other");
ClassifyP->NotifyClient           = ETryllNotifyClient::Always;
ClassifyP->FoundExit              = TEXT("pick");
ClassifyP->NotFoundExit           = TEXT("compose");

UTryllIntentToInstructionParams* PickP =
    UTryllNodeParamsFactory::MakeIntentToInstructionParams(this);
PickP->InlineKeys = { TEXT("insult"), TEXT("pleasing"), TEXT("joking") };
PickP->InlineStrings = {
    TEXT("The traveller has insulted you. Respond curtly and insult them back — short and sharp."),
    TEXT("The traveller is being polite. Stay grumpy but show grudging satisfaction. One sentence."),
    TEXT("The traveller told a joke. Permit yourself one brief laugh then refocus on duty."),
};
PickP->bOverrideOutputName = true;
PickP->OutputName          = TEXT("tone");
PickP->DefaultExit         = TEXT("compose");

UTryllTransformParams* ComposeP = UTryllNodeParamsFactory::MakeTransformParams(this);
ComposeP->bOverrideTemplate = true;
ComposeP->Template = TEXT("{{#instructions}}[Tone directive: {{text}}]\n{{/instructions}}Traveller says: {{user_message}}");
ComposeP->DefaultExit = TEXT("respond");

UTryllGenerateParams* RespondP = UTryllNodeParamsFactory::MakeGenerateParams(this);
RespondP->bOverrideSystemPrompt = true;
RespondP->SystemPrompt = TEXT("You are roleplaying as Aldric, a grumpy gate guard. Stay in character at all times. Use short sentences.");
RespondP->bOverrideInput = true;
RespondP->Input = TEXT("compose");
// RespondP->DefaultExit is TEXT("") (END) by default.

FTryllGraphDescription Graph = FTryllGraphBuilder()
    .AddNode(TEXT("classify"), ClassifyP)
    .AddNode(TEXT("pick"),     PickP)
    .AddNode(TEXT("compose"),  ComposeP)
    .AddNode(TEXT("respond"),  RespondP)
    .SetStartNode(TEXT("classify"))
    .SetDefaultModelName(TEXT("Llama 3.2 3B Instruct (Q4_K_M)"))
    .Build();
```

Or author it in the graph editor

Both editors expose every parameter above, including a multi-line `system_prompt` field and a registered-model dropdown for `model_name`. See [Edit workflows in the Unity graph editor](https://docs.tryllengine.com/how-to/edit-workflows-in-graph-editor.html) or [the Unreal equivalent](https://docs.tryllengine.com/how-to/edit-workflows-in-unreal-graph-editor.html).

______________________________________________________________________

## Step 4 — send messages

```
# insult → A → found → pick → respond (insult directive)
print(agent.send_message("Move aside, you dull-witted oaf."))

# pleasing → B → found → pick → respond (polite directive)
print(agent.send_message("Good day to you, honourable guard. May I pass?"))

# joking → C → found → pick → respond (joke directive)
print(agent.send_message("Why did the troll apply to be a guard? He heard it was rocky!"))

# other → D → not_found → compose → respond (no directive)
print(agent.send_message("How far is it to the next town?"))
```

______________________________________________________________________

## Step 5 — read the decision and tune the threshold

Because `notify_client=Always`, every turn pushes an `intent_llm_classified` event carrying `intent`, `top_prob`, `second_prob`, `not_found_reason`, and one `<label>.prob` per intent. Subscribe with `set_on_intent_llm_classified` / `SetOnIntentLlmClassified` / `IntentLlmClassified` / `OnIntentLlmClassified`, or watch it in the [Agent Log](https://docs.tryllengine.com/how-to/use-agent-log.html).

With `enable_diagnostics` on, `debug_info.nodes[]` carries the full picture for the `classify` node — including the rendered prompt, so this is where you confirm `{{intents_block}}` actually substituted:

```
{
  "parameters": { "threshold": 0.5, "margin": 0.15,
                  "intents_ids": ["insult", "pleasing", "joking", "other"] },
  "output": {
    "classification": {
      "query":            "Move aside, you dull-witted oaf.",
      "probs":            [0.81, 0.04, 0.07, 0.08],
      "top_prob":         0.81,
      "second_prob":      0.08,
      "margin_result":    0.73,
      "predicted_letter": "A",
      "intent":           "insult"
    }
  }
}
```

`probs[]` is positionally aligned with `parameters.intents_ids[]`, so you can read the whole ranking, not just the winner.

Do not ship with `threshold` at 0.5

Collect `top_prob` across a few dozen representative lines, then set `threshold` from that distribution. On the immersion-guard corpus this single change beat every prompt edit combined — the method is in [Tuning the threshold](https://docs.tryllengine.com/how-to/build-an-immersion-guard.html#tuning-the-threshold). `threshold` and `margin` are mutable at runtime ([Change agent parameters](https://docs.tryllengine.com/how-to/change-agent-parameters.html)), so you can sweep them on a live agent.

______________________________________________________________________

## Common pitfalls

- **Empty `system_prompt`.** There is no built-in default. The model gets an empty system message, never sees the labels, and picks a letter close to arbitrarily — with no error anywhere. This is the most common way to get nonsense out of this node.
- **`{{intents_block}}` missing from the template.** Same outcome as above, even though the rest of your prompt looks fine.
- **A comma inside a description.** Splits into an extra label; surfaces as `intents_ids count (4) does not match intents_prompt count (5)`.
- **Overlapping descriptions.** Two labels that could both describe one line split the probability mass, and the `margin` gate rejects it. Add a negation clause (`… — not case questions`) rather than lowering `margin`.
- **`not_found_intent` not in `intents_ids`.** `CreateAgent` fails.
- **Expecting `Generate`'s Mustache variables.** `{{user_message}}`, `{{slot.x}}` and `{{var.x}}` do not exist in a classifier system prompt.
- **Structural params.** `intents_ids`, `intents_prompt`, `model_name` and `input` are fixed for the agent's lifetime — `ChangeAgentParam` rejects them with `ParamNotMutable`. `system_prompt`, `history_turns`, `threshold`, `margin`, `notify_client` and `not_found_intent` are mutable.
- **Too many labels.** Past roughly 10–15 the single-pass approach degrades; use the embedding node, or pre-filter.

______________________________________________________________________

## Next steps

- **Add an intent** — append to both lists (catch-all stays last) and add a key/value pair to `pick`. No re-indexing, no files.
- **Gate before the character** — put a classifier in *front* of the NPC to drop out-of-character lines: [Build an immersion guard](https://docs.tryllengine.com/how-to/build-an-immersion-guard.html).
- **Branch instead of instructing** — route on the intent with a [`Branch`](https://docs.tryllengine.com/reference/nodes/branch.html) node when different intents need different *graphs*, not just different tones.
- **Fall through to retrieval** — wire `not_found` to a [`Retrieve`](https://docs.tryllengine.com/reference/nodes/retrieve.html) chain so confident lines get scripted beats and everything else gets a grounded answer.

______________________________________________________________________

## Related

- [How-to: Design an intent-classifier prompt](https://docs.tryllengine.com/how-to/design-an-intent-classifier-prompt.html) — the prompt-craft companion to this recipe
- [How-to: Build an intent-driven NPC](https://docs.tryllengine.com/how-to/build-intent-driven-npc.html) — the embedding-based version of this same NPC
- [Reference: Classify Intent (LLM) node](https://docs.tryllengine.com/reference/nodes/classify-intent-llm.html)
- [Reference: Intent to Instruction node](https://docs.tryllengine.com/reference/nodes/intent-to-instruction.html)
- [How-to: Use Mustache templates](https://docs.tryllengine.com/how-to/use-mustache-templates.html) — the `compose` node's template context
- [Concept: Slots and inter-node value passing](https://docs.tryllengine.com/concepts/slots-and-inter-node-value-passing.html)
