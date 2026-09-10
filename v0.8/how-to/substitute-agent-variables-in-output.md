# Substitute Agent Variables in LLM Output

Make the model emit a placeholder such as `__PRICE__` instead of inventing a number, then have the server replace that marker with a live [Agent Variable](https://docs.tryllengine.com/v0.8/concepts/agent-variables.html) value before the answer reaches the client, the dialogue history, or TTS.

This is **output-side** substitution. It is distinct from input-side Mustache templating (`{{var.price}}` in a prompt template), which injects values *before* generation. Use output substitution when the model should choose *where* a value appears in a sentence it writes this turn, without ever seeing the real number.

Prerequisites

- Per-agent variables declared at `CreateAgent` (see [Drive Prompts and Retrieval from Game State](https://docs.tryllengine.com/v0.8/how-to/drive-prompts-and-retrieval-from-game-state.html)).
- A `Generate` or `GenerateAndSpeak` node with `substitute_agent_variables` enabled.
- Prompt instructions (and ideally a GBNF grammar) that force the model to write `__NAME__` markers instead of raw values.

______________________________________________________________________

## 1. Mark variables as substitutable

Only variables with `allow_output_substitution = true` participate. Default is `false`. Sets (`set<string>`) cannot enable the flag. Substitutable string values are capped at 2048 UTF-8 bytes. Names must be marker-safe: no `__` substring and must not end with `_`.

On the workflow asset **Variables** row: tick **Allow Output Substitution** for scalar variables (`Int` / `Float` / `String` / `Bool`). The checkbox is disabled for `StringSet`. Component value overrides cannot change this permission.

```
from tryll_client.variables import VariableDecl

agent = client.create_agent(
    graph,
    variables={
        "price": VariableDecl(18, allow_output_substitution=True),
        "mood": "cheerful",  # plain value → not substitutable
    },
)
```

```
std::vector<Tryll::Client::AgentVariableDecl> variables = {
    {"price", std::int64_t{18}, /*allowOutputSubstitution=*/true},
};
```

______________________________________________________________________

## 2. Enable the node toggle

On the `Generate` or `GenerateAndSpeak` node, set `substitute_agent_variables = true` (mutable; default `false`). Matching is ASCII case-insensitive: `__PRICE__`, `__price__`, and `__PrIcE__` all resolve to the same declared name.

Instruct the model in the system prompt / template, for example:

```
When stating a price, write __PRICE__ instead of a number.
```

For deterministic QA or production barks, constrain the line with a GBNF `grammar` that includes the marker literally.

______________________________________________________________________

## 3. What happens at runtime

As tokens stream:

1. A small state machine buffers any possible `__…__` prefix across chunk boundaries.
1. Completed known markers are replaced with the rendered variable value (same formatting as `{{var.*}}`).
1. Unknown markers and incomplete markers at end-of-stream are left unchanged.
1. Inserted values are **not** rescanned (non-recursive).
1. Transformed text is written to the output slot, streamed/whole wire answer, history, and — for `GenerateAndSpeak` — TTS input.

On cancel, a buffered possible-marker prefix is discarded so clients/TTS never see a half-marker.

Update the variable between turns with the usual deferred-flush API (`SetInt("price", 42)` / `set_variables`); the next Generate picks up the new value.

______________________________________________________________________

## 4. Diagnostics

When `enable_diagnostics` is on, each Generate / GenerateAndSpeak node entry includes a typed `variable_replacements` array:

```
{
  "variable": "price",
  "matched": "__PrIcE__",
  "replacement": "18",
  "source_byte_offset": 11,
  "output_byte_offset": 11
}
```

Offsets are UTF-8 byte indices. At most 256 records are kept (`variable_replacements_truncated` when exceeded).

______________________________________________________________________

## When not to use this

- Use input-side `{{var.price}}` when the model must **reason about** the value—for example, compare it with another amount—and exact reproduction in the answer is not the main guarantee. Knowing a value before the turn does not by itself make input-side templating preferable: once the model sees a number, it can still round, alter, or omit it.
- Use output substitution when the value must appear **exactly as supplied** while the model controls only where it belongs in the sentence.
- If the value must also drive game logic (commit a sale), prefer a `ToolCall` structured move, then narrate — substitution only fixes text, not claims.

For a complete pricing-policy and transaction-safety example, see [Keep Merchant Prices Deterministic](https://docs.tryllengine.com/v0.8/how-to/keep-merchant-prices-deterministic.html). To strip speaker labels and roleplay markup before substitution runs, see [Filter LLM Output Artifacts](https://docs.tryllengine.com/v0.8/how-to/filter-llm-output-artifacts.html).
