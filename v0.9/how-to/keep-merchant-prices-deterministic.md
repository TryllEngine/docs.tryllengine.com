# Keep Merchant Prices Deterministic

Let game code calculate a merchant's price while the language model decides how to say it in character. The model emits `__PRICE__`; Tryll replaces that marker with the authoritative value before the text reaches the client, dialogue history, or TTS.

This pattern is called **Placeholder Post-Processing (PPP)** in recent game-trading research. It separates pricing policy from language generation: the game is the cashier, and the model is the merchant's voice.

What this guide guarantees

Output substitution guarantees that a recognised marker is replaced with the value supplied by the game. It does **not** validate inventory, approve a discount, or commit a transaction. Keep those decisions in deterministic game logic.

Prerequisites

- Read [Agent Variables](https://docs.tryllengine.com/v0.9/concepts/agent-variables.html).
- Follow [Substitute Agent Variables in LLM Output](https://docs.tryllengine.com/v0.9/how-to/substitute-agent-variables-in-output.html) for the complete substitution contract and validation rules.
- Use a `Generate` or `GenerateAndSpeak` node.

______________________________________________________________________

## The pattern

```
flowchart LR
    input["Player request"]
    policy["Game pricing policy<br>inventory, quantity, discounts"]
    variable["Agent variable<br>price = 18"]
    model["Generate / GenerateAndSpeak<br>&quot;For you, __PRICE__ gold.&quot;"]
    output["Authoritative output<br>&quot;For you, 18 gold.&quot;"]

    input --> policy
    policy --> variable
    variable --> model
    model --> output
```

The model never needs to copy, round, or calculate the amount. It only chooses where the marker belongs in a natural sentence.

______________________________________________________________________

## 1. Calculate the quote in game code

Put every rule that can change the transaction in one authoritative pricing function: item cost, quantity, reputation, bundle discounts, reservation price, tax, and currency conversion.

For example:

```
def calculate_quote(*, unit_price: int, quantity: int,
                    reputation_discount: float, minimum_total: int) -> int:
    subtotal = unit_price * quantity
    discounted = round(subtotal * (1.0 - reputation_discount))
    return max(minimum_total, discounted)
```

Treat the result as transaction state, not presentation text. Store it with the item ID and quantity so the eventual purchase can be revalidated against the same quote.

Do not ask the model to multiply quantities, apply discounts, enforce a price floor, or recover the authoritative amount from conversation history.

______________________________________________________________________

## 2. Declare `price` as substitutable

Declare the variable when creating the agent and opt it into output substitution. This permission belongs to the declaration and cannot be enabled by a component override later.

On the workflow asset, add a scalar variable named `price`, then enable **Allow Output Substitution**. Use `Int` for whole currency units or `String` when the game supplies a localized spoken form.

```
from tryll_client.variables import VariableDecl

agent = client.create_agent(
    graph,
    variables={
        "price": VariableDecl(
            0,
            allow_output_substitution=True,
        ),
    },
)
```

```
std::vector<Tryll::Client::AgentVariableDecl> variables = {
    {"price", std::int64_t{0}, /*allowOutputSubstitution=*/true},
};

auto agent = client.CreateAgent(
    graph,
    /*enableDiagnostics=*/false,
    std::nullopt,
    /*maintainDialogueHistory=*/true,
    variables);
```

Use separate variables when one line contains independently authoritative values, such as `__UNIT_PRICE__`, `__QUANTITY__`, and `__TOTAL__`. Each one must be declared and explicitly allowed.

______________________________________________________________________

## 3. Configure the merchant response

Enable `substitute_agent_variables` on the `Generate` or `GenerateAndSpeak` node. Then tell the model exactly when and how to emit the marker:

```
You are Branna, a guarded but witty weapons merchant.

When stating the current total price, write exactly __PRICE__ where the amount
belongs. Do not calculate, spell out, round, or replace the marker yourself.
Do not state any other total price. Keep the reply to one sentence.
```

`__PRICE__` matching is ASCII case-insensitive, but using one canonical spelling in prompts makes failures easier to diagnose.

For a fixed set of merchant barks, a [GBNF grammar](https://docs.tryllengine.com/v0.9/how-to/constrain-output-with-a-grammar.html) can require the marker literally. For open-ended negotiation, keep the grammar broad enough that it does not remove the language variation you wanted from the model.

With `GenerateAndSpeak`, substitution happens in the streamed server pipeline **before synthesis**. The player hears the replacement value, not the marker, without waiting for the complete generated line.

______________________________________________________________________

## 4. Set the quote before generation

Update the local variable mirror immediately before sending the player turn. The client flushes the pending write before `SendMessage`, so the same turn sees the new value.

```
long quote = CalculateQuote(item, quantity, player.Reputation);
agentComp.Agent.Variables.SetInt("price", quote);
agentComp.SendMessage(playerText);
```

```
const int64 Quote = CalculateQuote(Item, Quantity, PlayerReputation);
Agent->GetVariables().SetInt(TEXT("price"), Quote);
Agent->SendMessage(PlayerText);
```

```
const std::int64_t quote = CalculateQuote(item, quantity, reputation);
agent.Variables().SetInt("price", quote);
agent.SendText(playerText);
```

```
quote = calculate_quote(
    unit_price=item.unit_price,
    quantity=quantity,
    reputation_discount=discount,
    minimum_total=item.minimum_total,
)
agent.variables.set("price", quote)
reply = agent.send_message(player_text)
```

This path works when gameplay already knows the relevant item and quantity—for example, the player selected an item in shop UI before speaking.

### If the player's words determine the quote

When the game must first extract an item, quantity, or offer from the same utterance, use this graph:

```
flowchart LR
    detect["ToolCall<br>extract item / quantity / offer<br>disposition=Pause"]
    game["Game validates input<br>and calculates quote"]
    update["Set price variable"]
    speak["Resume → GenerateAndSpeak<br>with __PRICE__"]

    detect --> game
    game --> update
    update --> speak
```

While the turn is paused, update `price` and then call `Resume`. Pending variable writes flush before resume, so the downstream generation uses the newly calculated value in the same turn. See [Pause and Resume a Turn](https://docs.tryllengine.com/v0.9/how-to/pause-and-resume-turns.html) and [Define and Handle Tool Calls](https://docs.tryllengine.com/v0.9/how-to/define-and-handle-tool-calls.html).

Never trust the extracted arguments without validation. Check the tool name, item ID, quantity bounds, current inventory, and allowed state transition before calculating the quote.

______________________________________________________________________

## 5. Commit from game state, not narrated text

A correct spoken price is not transaction authorization. The model can still make an unsupported claim such as “I will include a free dagger.”

For a robust merchant:

1. Keep inventory, currency, quote, and negotiation state in the game.
1. Require an explicit confirmation step before purchase.
1. Recheck the item, quantity, current funds, inventory, and quoted amount.
1. Execute the transfer exactly once through game code or a validated tool call.
1. Treat the generated sentence as presentation only.

Do not parse the substituted answer to discover the price again. You already have the authoritative value that produced it.

______________________________________________________________________

## 6. Make spoken prices unambiguous

An integer such as `18` is usually sufficient for fantasy currency:

```
"For you, __PRICE__ gold pieces."
```

For localized currency, ranges, decimals, or values whose pronunciation matters, declare a substitutable string and let the game provide the final spoken form:

```
price_spoken = "eighteen gold pieces"
```

Then make the marker own the complete phrase:

```
"For you, __PRICE_SPOKEN__."
```

This prevents duplicated units and avoids depending on a TTS normalizer to infer the intended currency or locale. Substitutable strings are limited to 2048 UTF-8 bytes.

______________________________________________________________________

## Verify the result

Test at least these cases:

- Change the quote between turns and confirm the next response uses the new value.
- Use a mixed-case marker such as `__PrIcE__` and confirm it still resolves.
- Confirm the wire text, dialogue history, and generated speech all contain the replacement rather than the marker.
- Cancel during a partially generated marker and confirm no half-marker reaches the client or TTS.
- Attempt a below-floor price and verify game policy rejects it before updating the variable.
- Attempt to confirm a stale quote after inventory or player funds change and verify the transaction is revalidated.

With agent diagnostics enabled, inspect `variable_replacements` on the Generate or GenerateAndSpeak node to confirm the matched marker, replacement, and UTF-8 byte offsets.

______________________________________________________________________

## Why this pattern

The design follows a recurring result in negotiation and game-NPC research: separate deterministic strategy and state from neural language generation.

- [Decoupling Strategy and Generation in Negotiation Dialogues (He et al., 2018)](https://aclanthology.org/D18-1256/) established the manager/generator split: a controlled strategy chooses a move such as `propose(price=50)`, and a language model renders it.
- [State-Inference-Based Prompting for Natural Language Trading with Game NPCs (2025)](https://arxiv.org/pdf/2507.07203) uses placeholder-based price calculation and reports 99.7% calculation precision across its 100-dialogue evaluation.
- [Aligning Large Language Models with Procedural Rules: Autoregressive State-Tracking Prompting for In-Game Trading (2025)](https://arxiv.org/html/2510.25014v1) names the technique **Placeholder Post-Processing** and reports 99.3% calculation precision across 300 trading dialogues. In that evaluation, the scaffold also reduced response time from 21.2 seconds to 2.4 seconds when moving from the compared larger model to the smaller model.
- [Leveraging LLMs for Active Merchant NPCs (MART, 2024)](https://arxiv.org/html/2412.11189v2) documents the pricing, arithmetic, giveaway, and hallucinated-inventory problems that deterministic policy and validated actions must address.

These are results from the papers' own models, datasets, and experimental settings—not performance guarantees for a Tryll workflow. Their reusable engineering lesson is the boundary: let the model produce expressive language, but do not let it own a value that has one correct answer.

______________________________________________________________________

## Related

- [Substitute Agent Variables in LLM Output](https://docs.tryllengine.com/v0.9/how-to/substitute-agent-variables-in-output.html)
- [Agent Variables](https://docs.tryllengine.com/v0.9/concepts/agent-variables.html)
- [Pause and Resume a Turn](https://docs.tryllengine.com/v0.9/how-to/pause-and-resume-turns.html)
- [Define and Handle Tool Calls](https://docs.tryllengine.com/v0.9/how-to/define-and-handle-tool-calls.html)
- [Generate and Speak reference](https://docs.tryllengine.com/v0.9/reference/nodes/generate-and-speak.html)
