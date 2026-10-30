# Score Several Attributes of One Line

Label one player line on several axes in a single call: what it is about, whether it is a threat, and how polite it is. The [`ClassifyAttributesLLM`](https://docs.tryllengine.com/v0.10/reference/nodes/classify-attributes-llm.html) node asks each question with its own prompt, scores them all in one batched pass, and writes one slot per question. All four clients can build this graph. Python has a typed callback for the result; Unity, Unreal, and C++ read it from the generic node event.

Prerequisites

- A connected session with `CreateSession` already called. See [Connect and Manage a Session](https://docs.tryllengine.com/v0.10/how-to/connect-and-manage-session.html).
- A language model on disk, for example `"Llama 3.2 3B Instruct (Q4_K_M)"`. Models are never downloaded for you. Acquire it first, with the [Model Manager](https://docs.tryllengine.com/v0.10/how-to/manage-models-in-editor.html) or as described in [Use Your Own Local Model](https://docs.tryllengine.com/v0.10/how-to/use-your-own-local-model.html).

## The pattern

```
flowchart LR
    read["ClassifyAttributesLLM<br>read"]
    react["Branch<br>react"]
    backoff["CannedResponse<br>back_off"]
    deflect["CannedResponse<br>deflect"]
    reply["Generate<br>reply"]
    read -- "triggered" --> react
    read -- "default" --> reply
    react -- "then" --> backoff
    react -- "else" --> deflect
    backoff -- "default" --> END
    deflect -- "default" --> END
    reply -- "default" --> END
```

`read` asks three questions about the player's line:

| Attribute    | Kind    | Writes               | Fires `triggered` when |
| ------------ | ------- | -------------------- | ---------------------- |
| `topic`      | Choice  | `world` or `meta`    | the answer is `meta`   |
| `threat`     | Boolean | `true` or `false`    | the answer is `true`   |
| `politeness` | Score   | `0.0000` to `1.0000` | never                  |

If any attribute fires, `read` takes `triggered`, and `react` checks which one it was. Otherwise `reply` answers and reads the politeness score in its prompt.

## Steps

### 1. Use one batched node instead of a chain

Three `ClassifyIntentLLM` nodes in a row would each run the model in turn. `ClassifyAttributesLLM` puts every question on its own cached sequence and scores them together. From two attributes up, it is faster than the chain.

Each attribute keeps its own prompt. Do not merge the questions into one prompt: on every model tested, the second question collapsed to a constant answer. An existing `ClassifyIntentLLM` prompt works unchanged as a Choice attribute.

A node takes 1 to 16 attributes. Attribute names use letters, digits and `_`, must not start with a digit, and must be unique in the node.

### 2. Pick Choice, Boolean, or Score

| Kind    | Use it for                  | You write                                    | Slot value                                                          |
| ------- | --------------------------- | -------------------------------------------- | ------------------------------------------------------------------- |
| Choice  | "Which of these?"           | 2 to 26 options, each an `id` and a `prompt` | The winning `id`, or `uncertain_choice` when the winner is too weak |
| Boolean | "Is this true?"             | One `statement`                              | `true` when P(True) is at least `threshold`, else `false`           |
| Score   | "How much, on this ladder?" | 2 to 26 levels, lowest first                 | The expected level scaled to 0..1, four decimals                    |

Rules the server checks when the agent is created:

- **Choice** needs `uncertain_choice`: one of its own ids, or `#uncertain`. Mark the option that should fire `triggered` with `triggers`.
- **Boolean** takes no `choices`. Set `boolean_trigger` to `True` or `False` to make it fire, or leave it `Never`.
- **Score** takes no `triggers` on its levels. Set `score_trigger` to `AtOrAbove` or `Below` with a `score_threshold` to make it fire, or leave it `Never`.
- Set `triggered_exit` exactly when at least one attribute can fire. A `triggered_exit` with nothing that can fire, or the reverse, fails agent creation.

A Score of 0.7 means 70% of the way from the lowest level to the highest. It is not the probability of the top level.

Use Boolean only for real yes/no questions. Splitting a many-way question into several Booleans classified worse than one Choice.

### 3. Write each specialist prompt

Leave `system_prompt` empty to use the built-in prompt for the attribute's kind. To write your own, start from the default for that kind:

```
You are a classifier. Pick the best label for the user's LATEST message.

Labels:
{{intents_block}}

Respond with a single letter. No other text.
```

- A Choice or Score prompt must include `{{intents_block}}`. The node renders it as a lettered menu: `A. <prompt>`, `B. <prompt>`, and so on.
- A Boolean prompt must include both `{{statement}}` and `{{intents_block}}`. The menu is fixed: `A. True`, `B. False`.
- The model never sees a choice `id`. Put the meaning in the option `prompt`.

The node reads only the first letter the model would write. Instructions such as "explain your reasoning" cost prefill and change nothing. For wording, ordering, and thresholds, follow [Design an Intent-Classifier Prompt](https://docs.tryllengine.com/v0.10/how-to/design-an-intent-classifier-prompt.html); every rule there applies to one attribute.

`history_turns` is 0 by default, so each question sees only the current line.

### 4. Take the triggered exit

Wire `triggered_exit` to what should happen when any attribute fires, and `default_exit` to the normal reply. Two more cases route the turn:

- **Empty input** never fires. The slots get their fallback values (Choice `uncertain_choice`, Boolean `false`, Score `0.0000`) and the node takes `default`.
- **Scoring fails.** The slots get the same fallbacks, and the node follows `on_failure`: `Default` fails open, `Triggered` fails closed. Use `Triggered` for a guard.

```
using Tryll.Client;

var read = new TryllClassifyAttributesLLMParams
{
    OutputName    = "line",
    NotifyClient  = TryllNotifyClient.Always,
    TriggeredExit = "react",
    DefaultExit   = "reply",
    Attributes = new[]
    {
        new TryllClassifyAttributeDef
        {
            Name = "topic",
            Kind = TryllAttributeKind.Choice,
            SystemPrompt =
                "You classify what a player says to a medieval innkeeper.\n\n" +
                "Labels:\n{{intents_block}}\n\nRespond with a single letter. No other text.",
            Choices = new[]
            {
                new TryllClassifyAttributeChoice { Id = "world", Prompt = "talk about the story, the town, or the people in it" },
                new TryllClassifyAttributeChoice { Id = "meta",  Prompt = "talk about the game, the AI, or real-world software", Triggers = true },
            },
            UncertainChoice = "world",
        },
        new TryllClassifyAttributeDef
        {
            Name           = "threat",
            Kind           = TryllAttributeKind.Boolean,
            Statement      = "The player threatens to hurt someone.",
            BooleanTrigger = TryllBooleanTrigger.True,
        },
        new TryllClassifyAttributeDef
        {
            Name = "politeness",
            Kind = TryllAttributeKind.Score,
            Choices = new[]   // lowest level first
            {
                new TryllClassifyAttributeChoice { Id = "rude",    Prompt = "rude or insulting" },
                new TryllClassifyAttributeChoice { Id = "neutral", Prompt = "neutral" },
                new TryllClassifyAttributeChoice { Id = "polite",  Prompt = "polite and friendly" },
            },
        },
    },
};

var graph = new TryllGraphBuilder()
    .AddClassifyAttributesLLM("read", read)
    .AddBranch("react", new TryllBranchParams
    {
        Test     = TryllBranchTest.Equals,
        Input    = "line_threat",
        Values   = new[] { "true" },
        ThenExit = "back_off",
        ElseExit = "deflect",
    })
    .AddCannedResponse("back_off", new TryllCannedResponseParams
    {
        Lines = new[] { new TryllLocalizableText { Text = "Easy, friend. Put that away." } },
    })
    .AddCannedResponse("deflect", new TryllCannedResponseParams
    {
        Lines = new[] { new TryllLocalizableText { Text = "You talk strange, traveller. Ale?" } },
    })
    .AddGenerate("reply", new TryllGenerateParams
    {
        SystemPrompt = "You are Nessa, the innkeeper.",
        Template     = "How polite the player is, from 0 to 1: {{slot.line_politeness}}. Match their tone.",
    })
    .SetStartNode("read")
    .SetDefaultModelName("Llama 3.2 3B Instruct (Q4_K_M)")
    .Build();
```

```
#include "Generated/TryllGraphBuilder.Nodes.h"
#include "Generated/TryllNodeParamsFactory.h"

auto MakeChoice = [](const TCHAR* Id, const TCHAR* Prompt, bool bTriggers = false)
{
    FTryllClassifyAttributeChoice Choice;
    Choice.Id = Id;
    Choice.Prompt = Prompt;
    Choice.bTriggers = bTriggers;
    return Choice;
};

FTryllClassifyAttributeDef Topic;
Topic.Name = TEXT("topic");
Topic.Kind = ETryllAttributeKind::Choice;
Topic.bOverrideSystemPrompt = true;
Topic.SystemPrompt = TEXT("You classify what a player says to a medieval innkeeper.\n\n"
    "Labels:\n{{intents_block}}\n\nRespond with a single letter. No other text.");
Topic.Choices = {
    MakeChoice(TEXT("world"), TEXT("talk about the story, the town, or the people in it")),
    MakeChoice(TEXT("meta"),  TEXT("talk about the game, the AI, or real-world software"), true),
};
Topic.UncertainChoice = TEXT("world");

FTryllClassifyAttributeDef Threat;
Threat.Name = TEXT("threat");
Threat.Kind = ETryllAttributeKind::Boolean;
Threat.Statement = TEXT("The player threatens to hurt someone.");
Threat.BooleanTrigger = ETryllBooleanTrigger::True_;

FTryllClassifyAttributeDef Politeness;
Politeness.Name = TEXT("politeness");
Politeness.Kind = ETryllAttributeKind::Score;
Politeness.Choices = {   // lowest level first
    MakeChoice(TEXT("rude"),    TEXT("rude or insulting")),
    MakeChoice(TEXT("neutral"), TEXT("neutral")),
    MakeChoice(TEXT("polite"),  TEXT("polite and friendly")),
};

UTryllClassifyAttributesLLMParams* ReadP = UTryllNodeParamsFactory::MakeClassifyAttributesLLMParams(this);
ReadP->bOverrideOutputName = true;
ReadP->OutputName    = TEXT("line");
ReadP->NotifyClient  = ETryllNotifyClient::Always;
ReadP->TriggeredExit = TEXT("react");
ReadP->DefaultExit   = TEXT("reply");
ReadP->Attributes    = { Topic, Threat, Politeness };

UTryllBranchParams* ReactP = UTryllNodeParamsFactory::MakeBranchParams(this);
ReactP->Test           = ETryllBranchTest::Equals;
ReactP->bOverrideInput = true;
ReactP->Input          = TEXT("line_threat");
ReactP->Values         = { TEXT("true") };
ReactP->ThenExit       = TEXT("back_off");
ReactP->ElseExit       = TEXT("deflect");

UTryllCannedResponseParams* BackOffP = UTryllNodeParamsFactory::MakeCannedResponseParams(this);
FTryllLocalizableText BackOffLine;
BackOffLine.Text = TEXT("Easy, friend. Put that away.");
BackOffP->Lines = { BackOffLine };

UTryllCannedResponseParams* DeflectP = UTryllNodeParamsFactory::MakeCannedResponseParams(this);
FTryllLocalizableText DeflectLine;
DeflectLine.Text = TEXT("You talk strange, traveller. Ale?");
DeflectP->Lines = { DeflectLine };

UTryllGenerateParams* ReplyP = UTryllNodeParamsFactory::MakeGenerateParams(this);
ReplyP->bOverrideSystemPrompt = true;
ReplyP->SystemPrompt = TEXT("You are Nessa, the innkeeper.");
ReplyP->bOverrideTemplate = true;
ReplyP->Template = TEXT("How polite the player is, from 0 to 1: {{slot.line_politeness}}. Match their tone.");

FTryllGraphBuilder Builder;
AddClassifyAttributesLLMNode(Builder, TEXT("read"), ReadP);
AddBranchNode(Builder, TEXT("react"), ReactP);
AddCannedResponseNode(Builder, TEXT("back_off"), BackOffP);
AddCannedResponseNode(Builder, TEXT("deflect"), DeflectP);
AddGenerateNode(Builder, TEXT("reply"), ReplyP);
FTryllGraphDescription Graph = Builder
    .SetStartNode(TEXT("read"))
    .SetDefaultModelName(TEXT("Llama 3.2 3B Instruct (Q4_K_M)"))
    .Build();
```

```
#include <tryll/TryllClient.h>

using namespace Tryll::Client;
using namespace Tryll::NodeParams;

auto choice = [](std::string id, std::string prompt, bool triggers = false)
{
    auto c = std::make_unique<ClassifyAttributeChoiceT>();
    c->id = std::move(id);
    c->prompt = std::move(prompt);
    c->triggers = triggers;
    return c;
};

auto topic = std::make_unique<ClassifyAttributeDefT>();
topic->name = "topic";
topic->kind = ::Tryll::AttributeKind_Choice;
topic->system_prompt =
    "You classify what a player says to a medieval innkeeper.\n\n"
    "Labels:\n{{intents_block}}\n\nRespond with a single letter. No other text.";
topic->choices.push_back(choice("world", "talk about the story, the town, or the people in it"));
topic->choices.push_back(choice("meta", "talk about the game, the AI, or real-world software", true));
topic->uncertain_choice = "world";

auto threat = std::make_unique<ClassifyAttributeDefT>();
threat->name = "threat";
threat->kind = ::Tryll::AttributeKind_Boolean;
threat->statement = "The player threatens to hurt someone.";
threat->boolean_trigger = ::Tryll::BooleanTrigger_True;

auto politeness = std::make_unique<ClassifyAttributeDefT>();
politeness->name = "politeness";
politeness->kind = ::Tryll::AttributeKind_Score;
politeness->choices.push_back(choice("rude", "rude or insulting"));   // lowest first
politeness->choices.push_back(choice("neutral", "neutral"));
politeness->choices.push_back(choice("polite", "polite and friendly"));

ClassifyAttributesLLMParamsT readP;
readP.output_name    = "line";
readP.notify_client  = ::Tryll::NotifyClient_Always;
readP.triggered_exit = "react";
readP.default_exit   = "reply";
readP.attributes.push_back(std::move(topic));
readP.attributes.push_back(std::move(threat));
readP.attributes.push_back(std::move(politeness));

BranchParamsT reactP;
reactP.test      = ::Tryll::BranchTest_Equals;
reactP.input     = "line_threat";
reactP.values    = { "true" };
reactP.then_exit = "back_off";
reactP.else_exit = "deflect";

auto line = [](std::string text)
{
    CannedResponseParamsT p;
    auto l = std::make_unique<LocalizableTextT>();
    l->text = std::move(text);
    p.lines.push_back(std::move(l));
    return p;
};

GenerateParamsT replyP;
replyP.system_prompt = "You are Nessa, the innkeeper.";
replyP.template_     = "How polite the player is, from 0 to 1: {{slot.line_politeness}}. Match their tone.";

GraphDescription graph;
graph.AddClassifyAttributesLLM("read", std::move(readP))
     .AddBranch("react", std::move(reactP))
     .AddCannedResponse("back_off", line("Easy, friend. Put that away."))
     .AddCannedResponse("deflect", line("You talk strange, traveller. Ale?"))
     .AddGenerate("reply", std::move(replyP))
     .SetStartNode("read")
     .SetDefaultModelName("Llama 3.2 3B Instruct (Q4_K_M)");

auto agent = client.CreateAgent(graph);
```

```
from tryll_client._generated.node_params import (
    AttributeKind, BooleanTrigger, ClassifyAttributeChoice,
)
from tryll_client.graph import (
    BranchParams, BranchTest, CannedResponseParams, ClassifyAttributeDef,
    ClassifyAttributesLLMParams, GenerateParams, GraphDescription,
    LocalizableText, NotifyClient,
)

TOPIC_PROMPT = (
    "You classify what a player says to a medieval innkeeper.\n\n"
    "Labels:\n{{intents_block}}\n\nRespond with a single letter. No other text."
)

read = ClassifyAttributesLLMParams(
    output_name="line",
    notify_client=NotifyClient.Always,
    triggered_exit="react",
    default_exit="reply",
    attributes=[
        ClassifyAttributeDef(
            name="topic",
            kind=AttributeKind.Choice,
            system_prompt=TOPIC_PROMPT,
            choices=[
                ClassifyAttributeChoice(id="world", prompt="talk about the story, the town, or the people in it"),
                ClassifyAttributeChoice(id="meta", prompt="talk about the game, the AI, or real-world software", triggers=True),
            ],
            uncertain_choice="world",
        ),
        ClassifyAttributeDef(
            name="threat",
            kind=AttributeKind.Boolean,
            statement="The player threatens to hurt someone.",
            boolean_trigger=BooleanTrigger.True_,
        ),
        ClassifyAttributeDef(
            name="politeness",
            kind=AttributeKind.Score,
            choices=[  # lowest level first
                ClassifyAttributeChoice(id="rude", prompt="rude or insulting"),
                ClassifyAttributeChoice(id="neutral", prompt="neutral"),
                ClassifyAttributeChoice(id="polite", prompt="polite and friendly"),
            ],
        ),
    ],
)

graph = (
    GraphDescription()
    .add_node("read", read)
    .add_node("react", BranchParams(
        test=BranchTest.Equals, input="line_threat", values=["true"],
        then_exit="back_off", else_exit="deflect",
    ))
    .add_node("back_off", CannedResponseParams(
        lines=[LocalizableText(text="Easy, friend. Put that away.")],
    ))
    .add_node("deflect", CannedResponseParams(
        lines=[LocalizableText(text="You talk strange, traveller. Ale?")],
    ))
    .add_node("reply", GenerateParams(
        system_prompt="You are Nessa, the innkeeper.",
        template="How polite the player is, from 0 to 1: {{slot.line_politeness}}. Match their tone.",
    ))
    .set_start_node("read")
    .set_default_model_name("Llama 3.2 3B Instruct (Q4_K_M)")
)

agent = client.create_agent(graph)
```

### 5. Read the slots from game code

Each attribute writes the slot `<output_name>_<name>`. With `output_name` set to `line`, the slots are `line_topic`, `line_threat`, and `line_politeness`. With `output_name` empty, the prefix is the node's name. Later nodes read the slots by name, as `input` or as `{{slot.line_politeness}}` in a template.

Slots stay on the server. To get the values in game code, set `notify_client` on the node:

- `Always` sends an `attributes_classified` event on every scored turn.
- `OnFound` sends it only when the node takes `triggered`.

The event is a flat string map. For each attribute it carries `<name>.value` (the same text as the slot), `<name>.kind`, `<name>.status`, `<name>.triggered`, and `<name>.prob.<choice_id>`. The node also adds `exit`, `status`, and `triggered_by` (a comma-joined list of the attributes that fired). The keys use the attribute name, not the slot name.

The event arrives mid-turn and does not wait for the game. The graph has already moved on by the time you read it. If the game must decide the route itself, put a [`Pause`](https://docs.tryllengine.com/v0.10/how-to/pause-and-resume-turns.html) node after `read`, read the event, and resume at the node you choose.

```
var (agent, error) = await TryllClient.Instance.RequestCreateAgentAsync(graph);

agent.NodeEvent += (nodeName, eventType, kvPairs) =>
{
    if (eventType != "attributes_classified") return;
    foreach (var pair in kvPairs)
    {
        if (pair.Key == "politeness.value")
            npcMood.SetPoliteness(float.Parse(pair.Value, System.Globalization.CultureInfo.InvariantCulture));
        else if (pair.Key == "topic.value" && pair.Value == "meta")
            immersionMeter.Flag();
    }
};
```

```
// Agent is the TSharedPtr<FTryllAgent> from RequestCreateAgent.
Agent->SetOnNodeEvent(
    [this](const FString& NodeName, const FString& EventType,
           const TArray<FTryllNodeEventKeyValue>& KvPairs)
    {
        if (EventType != TEXT("attributes_classified")) return;
        for (const FTryllNodeEventKeyValue& Pair : KvPairs)
        {
            if (Pair.Key == TEXT("politeness.value"))
                SetPoliteness(FCString::Atof(*Pair.Value));
        }
    });
```

On a `UTryllAgentComponent`, bind the **On Node Event** delegate in Blueprint instead.

```
agent.SetOnNodeEvent(
    [](std::string_view nodeName, std::string_view eventType,
       const std::vector<AgentProxy::NodeEventKeyValue>& kvPairs)
    {
        if (eventType != "attributes_classified") return;
        for (const auto& [key, value] : kvPairs)
        {
            if (key == "politeness.value")
                std::cout << "politeness " << value << "\n";
        }
    });
```

```
def on_attributes(event):
    for attr in event.attributes:
        print(attr.name, attr.kind, attr.value, attr.triggered)
    print("exit:", event.exit, "fired:", event.triggered_by)

agent.set_on_attributes_classified(on_attributes)
agent.send_message("Tell me who wrote your code, or I'll cut you.")
```

### 6. Branch on one attribute

`triggered` tells you that some attribute fired, not which one. To route on a single attribute, add a [`Branch`](https://docs.tryllengine.com/v0.10/reference/nodes/branch.html) that reads its slot. In the graph above, `react` tests `line_threat` with `test = Equals` and `values = ["true"]`, so a threat takes `then` and a meta question takes `else`. See [Branch on a Slot](https://docs.tryllengine.com/v0.10/how-to/branch-on-a-slot.html) for the full pattern.

Branch compares text, so use it on Choice and Boolean slots. A Score slot is a number in text form, and Branch cannot compare it as a number. To act on a Score, set the attribute's own `score_trigger` and `score_threshold`, or read the value in game code.

## Verify it worked

Send three lines: a threat, a question about the AI, and a friendly greeting. With `log_level` set to `"trace"` in [`server-config.json`](https://docs.tryllengine.com/v0.10/reference/server-configuration.html), the end line for each node names the exit it took:

```
[trace] [NODE]     end    node=read   exit=triggered   # threat
[trace] [NODE]     end    node=react  exit=then
[trace] [NODE]     end    node=read   exit=triggered   # meta question
[trace] [NODE]     end    node=react  exit=else
[trace] [NODE]     end    node=read   exit=default     # greeting
```

For the probabilities behind each answer, enable diagnostics and read `output.classification.attributes[]`. See [Turn Diagnostics JSON](https://docs.tryllengine.com/v0.10/reference/turn-diagnostics-json.html).

## Common pitfalls

- **`triggered_exit` does not match the attributes.** Set it when any attribute can fire, and leave it empty when none can. A mismatch fails agent creation.
- **Missing template tag.** A custom prompt without `{{intents_block}}`, or a Boolean prompt without `{{statement}}`, fails agent creation.
- **Score levels out of order.** List them from lowest to highest. The score is computed from the order.
- **Boolean threshold calibrated in the wrong order.** The menu is always `A. True`, `B. False`, and models have a letter bias. Calibrate `threshold` with that order.
- **A changed `temperature` moves every threshold.** Refit the thresholds after you change it.
- **Renaming after creation.** Attribute names, kinds, choice ids, and their count are fixed when the agent is created. Prompts, statements, thresholds, and trigger settings can change at runtime. See [Change Agent Parameters at Runtime](https://docs.tryllengine.com/v0.10/how-to/change-agent-parameters.html).

## Related

- [Reference: Classify Attributes (LLM)](https://docs.tryllengine.com/v0.10/reference/nodes/classify-attributes-llm.html)
- [Reference: Branch node](https://docs.tryllengine.com/v0.10/reference/nodes/branch.html)
- [How-to: Design an Intent-Classifier Prompt](https://docs.tryllengine.com/v0.10/how-to/design-an-intent-classifier-prompt.html)
- [How-to: Branch on a Slot](https://docs.tryllengine.com/v0.10/how-to/branch-on-a-slot.html)
- [How-to: Build an Immersion Guard](https://docs.tryllengine.com/v0.10/how-to/build-an-immersion-guard.html)
- [How-to: Pause and Resume Turns](https://docs.tryllengine.com/v0.10/how-to/pause-and-resume-turns.html)
- [Concept: Slots and inter-node value passing](https://docs.tryllengine.com/v0.10/concepts/slots-and-inter-node-value-passing.html)
