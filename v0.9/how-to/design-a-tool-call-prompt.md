# Design a Tool-Call Prompt

How to write the tool schema and the `ToolCall` node's `system_prompt` so a small on-device model fills the right slots — and leaves the others alone.

The recipes come from iterating the Tryll-authored tool-call suites against the catalog (twenty models, greedy decoding). The finding that paid off was not cleverer prose. It was changing what the model is *asked to say*.

Tryll does not let you pick a tool-call format. The model's chat template renders the schema you declare. Your levers are the tool list, the parameter names and types, the descriptions, and the system prompt. A tool on the `ToolCall` node has a `name`, a `description`, and `parameters`; each parameter has only a `name`, a `type`, and a `description`. There is no `enum` field and no optional flag: the node marks every declared parameter as required in the schema it renders, so allowed values go in the parameter's `description`. See [Tool calling](https://docs.tryllengine.com/v0.9/concepts/tool-calling.html).

How claims are marked

- **Tested** — measured on the authored suites; the effect repeated across the catalog.
- **Hint** — real, but narrower: one suite, or a design rule rather than a score delta.
- **No effect / made it worse** — we tried it; do not spend time here.

______________________________________________________________________

## The failure that looks like a knowledge gap

Small models extract a filter the player *did* name very reliably. They fail at restraint. Asked "where did I see the orc?" with optional `region` / `event_type` / `time_window`, they invent a region rather than omit the argument — and they invent the **first allowed value listed**.

On the game-log suite that pattern was the bulk of the error: specified filters were right about 97% of the time; an unspecified `region` was wrong 63% of the time, and `Ironwood` (then the first region listed) beat the other three regions 51 to 7. That is slot-filling pressure, not geography.

The same pressure shows up as a guessed door (`left`, the first door listed) when the player said "lock the door" without saying which.

______________________________________________________________________

## Recipe 1 — name "unfiltered", put it first, require the slot

**Tested.** This is the change that moved the field.

Do not ask the model to *omit* a parameter. Give "the player did not narrow this down" a real value, list it first among the allowed values, and keep the parameter required — which the `ToolCall` node does for every declared parameter. The thing a pressured model reaches for becomes the right answer.

Across the catalog, adding `any` this way took the game-log suite from 46% to 77% exact accuracy. The three fields that gained `any` dropped from 44–63% wrong (when they should have been unfiltered) to roughly the error rate the models already had on filters they were told to set.

```
from tryll_client.graph import ToolDefinition, ToolParamDefinition

search_events = ToolDefinition(
    name="search_events",
    description="Find matching events in the activity log.",
    parameters=[
        ToolParamDefinition(name="entity", type="string",
            description='Who or what the question is about. '
                        'Pass "any" when the player did not name one.'),
        ToolParamDefinition(name="event_type", type="string",
            description='Kind of logged event: one of "any", "kill", "sighting", '
                        '"dialogue", "loot", "quest". '
                        'Pass "any" when the player did not imply one.'),
        ToolParamDefinition(name="region", type="string",
            description='Where it happened: one of "any", "Ironwood", "Riverwood", '
                        '"Karhold", "Gutter Ward". '
                        'Pass "any" when the player did not name a place.'),
        ToolParamDefinition(name="time_window", type="string",
            description='When it happened: one of "any", "today", "last_session", '
                        '"last_week". Pass "any" when the player did not give a time.'),
    ],
)
```

The node renders each tool for the chat template as a JSON schema with `type` and `description` per property and all four names in `required`.

The system prompt has to say the same thing the schema does:

```
Every call sets all four filters. When the player did not narrow one down,
pass `any`. That is the correct value, not a fallback — `any` means "do not
filter on this". Never guess a region, a time window, or a subject the player
did not mention.
```

The same pattern applies to actuation: lead the door and window value lists with `unspecified` (both parameters are required, like every declared parameter), and tell the servant that "unspecified" is the right answer when the master did not say which.

Your client then treats `any` / `unspecified` as "no filter / ask which one", instead of hoping the model leaves a JSON key out.

______________________________________________________________________

## Recipe 2 — do not add a parameter you can live without

**Tested. Made it worse.**

"Where did I *last* see the orc?" is not expressible if the tool has no "most recent" knob. Adding a required `sort` argument so that line could be answered made every model we re-ran *worse* — not only on the new slot, but on the four existing filters. Counts on four models: 19→18, 19→15, 18→12, 15→9 out of 20.

Each extra required parameter adds to the pressure to fill every slot. Four filters was the ceiling on that suite, not an arbitrary choice. Rename tricks did not help either: swapping `last_session` so the word "last" would stop pulling a time window changed nothing. A query with no "last" in it still drew a time filter.

If a distinction cannot be said without a fifth argument, prefer a slightly wrong query over a schema the model cannot fill. Keep the tool list short for the same reason — see [What makes tool calls reliable](https://docs.tryllengine.com/v0.9/concepts/tool-calling.html#what-makes-tool-calls-reliable-or-unreliable).

______________________________________________________________________

## Recipe 3 — do not forbid the call you want

**Tested.**

A prompt that says "never call a tool with every parameter unspecified" will be obeyed — including on the line whose correct call is exactly `operate_window(unspecified, unspecified)` ("Do something about that window"). Half the field answered in character instead of calling, which is what we asked for and then scored as a miss.

Write the exception where you mean it: greetings and questions get no call; a vague *order* still gets a call, with `unspecified` filled in. Put that in the system prompt next to the rule for `unspecified`, not as a blanket ban.

______________________________________________________________________

## Recipe 4 — split tools on intent, not on wording

**Hint.**

Three tools that sound alike need a rule the model can apply in one step:

- `propose_price` — they named a figure.
- `request_discount` — they want a lower price and named no figure.
- `refuse_deal` — they are walking away ("No deal.", "Forget it.").

"Not at that price, no." is not a refusal in that scheme — it is a discount request with no number. If you cannot say in one sentence why a line maps to exactly one tool, do not score it, and do not ship it as a distinct tool. Ambiguous splits train you to argue with the eval; they do not make the runtime call more reliable.

Parameterless classifier tools (`attack_player`, `start_trading`, `end_dialogue`) are a different job again. Models that extract arguments well still decline to fire a zero-argument tool and answer in character instead. That is capability, not missing instructions — every suite already tells the model when *not* to call. See [Compare tool-call models](https://docs.tryllengine.com/v0.9/how-to/compare-tool-call-models.html#pick-a-job-then-a-model) for which catalog models hold up on that flavour.

______________________________________________________________________

## Recipe 5 — descriptions are the prompt the model actually sees

**Hint** — this is also the concept-page advice; the eval did not contradict it.

The chat template inlines your `description` strings next to the JSON schema. "Pass `any` when the player did not name a place" on the `region` field does more than a long character biography in `system_prompt`. Keep the system prompt for routing (which tool, when to stay quiet) and put slot-filling rules on the slots.

Use `temperature = 0` on the `ToolCall` node. Sampling is not a meaningful lever here; both corpora move about a point from greedy to 0.7.

______________________________________________________________________

## What not to do

| Temptation                                                              | What happened                                                                 |
| ----------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| Declare a parameter with no sentinel and hope the model "just omits it" | Every declared parameter is required, and models fill the first listed value. |
| Put the sentinel last in the value list (`…, "unspecified"`)            | The guessed value is still the first one listed.                              |
| Add `sort` / extra filters so every English distinction is expressible  | Accuracy fell on the new slot *and* the old ones.                             |
| Rename values to dodge a word in the utterance                          | No change.                                                                    |
| Ban "all unspecified" in the prompt to stop chit-chat from calling      | It also stops the vague order you wanted.                                     |

Abstention ("Hello", "Nice weather") is mostly model quality. A clear do-not-call line in the prompt is necessary and not sufficient; weaker models still grab a tool whose argument matches a noun in the sentence.

______________________________________________________________________

## Related

- [Compare tool-call models](https://docs.tryllengine.com/v0.9/how-to/compare-tool-call-models.html)
- [Define and handle tool calls](https://docs.tryllengine.com/v0.9/how-to/define-and-handle-tool-calls.html)
- [Design an NPC prompt](https://docs.tryllengine.com/v0.9/how-to/design-an-npc-prompt.html) — the same "measure, then write" approach for spoken dialogue
- [Concept: Tool calling](https://docs.tryllengine.com/v0.9/concepts/tool-calling.html)
- [Reference: ToolCall node](https://docs.tryllengine.com/v0.9/reference/nodes/tool-call.html)
