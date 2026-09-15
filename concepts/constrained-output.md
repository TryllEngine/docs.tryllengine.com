# Constrained Output (GBNF)

Sometimes you don't want a small model to write *prose* — you want it to emit exactly one of a handful of machine-readable answers: a command (`cut red wire`), a dialogue choice, an emotion tag, a tiny JSON object. A [GBNF](https://github.com/ggml-org/llama.cpp/blob/master/grammars/README.md) **grammar** on a [Generate](https://docs.tryllengine.com/reference/nodes/generate.html) (or [Generate and Speak](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html)) node makes that guaranteed: set the `grammar` param and the model's output is forced to match the grammar at every step. This page explains what that does and — more importantly — what it does *not* do.

## What a grammar actually does

At every decode step the grammar is compiled to a state machine that knows which tokens could legally continue the output so far. The sampler sets the probability of every *other* token to −∞ and renormalises what's left; the model then picks from the surviving tokens as usual. Malformed output isn't "discouraged" — it is **unsampleable**.

```
root   ::= action " " object
action ::= "grab" | "pull" | "cut" | "activate"
object ::= "wire" | "enemy" | "lever" | "door"
```

With this grammar every reply is one of the 4×4 = 16 legal strings. No JSON braces to balance, no `"Sure! I think you meant…"` preamble to strip, no parse failures. For a weak model this can be the difference between "unusable" and "usable".

Two hard requirements

A grammar must contain a rule named `root` (that's the start symbol), and it must not be left-recursive. Tryll validates the grammar **up front**: an invalid grammar fails agent creation with [`GraphCompilationFailed`](https://docs.tryllengine.com/reference/error-codes.html), and an invalid grammar pushed later via [`ChangeAgentParam`](https://docs.tryllengine.com/how-to/change-agent-parameters.html) is rejected with [`InvalidParamValue` (3007)](https://docs.tryllengine.com/reference/error-codes.html) — you never discover a typo halfway through a turn.

## Syntax, never semantics

This is the one idea to internalise. **A grammar guarantees the *shape* of the output, never its *meaning*.** It removes illegal *options*; it has no idea what the player actually wanted.

Feed the grammar above a clean `"cut the wire"` and you'll get `cut wire`. Feed it a garbled `"cat the wiiire"` and the model will still emit a valid string — probably `cut wire`, because that's what its own weights find most likely among the allowed tokens. But feed it something the vocabulary can't express — `"open the door"` when `open` isn't in the action set — and it will confidently emit a **valid-but-wrong** command like `pull door`. The grammar did its job (the output is legal); the model guessed.

So constrained output is *neutral-to-helpful* for closed-set extraction (a verb, a slot, a choice) and a genuine trap if you assume validity implies correctness. When "did the player actually mean a command?" matters, gate it — see [below](#grammar-vs-tool-calling-vs-intent-classification).

## Constrain the wire, not the thinking

If your model is a reasoning model that emits a `<think>…</think>` block, a grammar that only permits `action object` forbids those thinking tokens outright — generation breaks or degrades badly. The rule of thumb from the research literature holds here: **constrain the final wire format, never the model's reasoning.** For command-parsing that means turning thinking off (the per-variant `disable_thinking` switch in the [model catalog](https://docs.tryllengine.com/reference/model-management.html)) so there's no chain-of-thought for the grammar to collide with. Extraction tasks don't need it anyway.

## The `unknown` / sentinel pattern

A closed grammar can't say "I didn't catch that" — every slot must be *something*. The usual fix is to make a sentinel like `unknown` a first-class option in each slot:

```
root    ::= action " " quality " " object
action  ::= "smash" | "grab" | "open" | "cut" | "unknown"
quality ::= "red" | "green" | "front" | "back" | "unknown"
object  ::= "wire" | "door" | "crate" | "safe" | "unknown"
```

Now the model *can* represent a missing or unrecognised slot (`cut unknown wire` when no colour was named). But here's the catch, and it's the same lesson as above: **the grammar makes `unknown` sampleable; it does not make the model *use* it.** Choosing `unknown` over guessing a plausible value is pure instruction-following, and how well a model does that varies a lot — and *not* simply with size. In our own measurements a strong 4B-class model followed the "emit `unknown` when unsure" rule almost perfectly, while another model of the *same size* essentially never did (it always guessed), and a larger model landed in between. The takeaways:

- **Spell the rule out** in the [system prompt](https://docs.tryllengine.com/reference/nodes/generate.html) — list the allowed words, the slot order, and explicitly "output `unknown` for any slot you can't fill".
- **Measure it on your actual model.** Don't assume the sentinel path works; feed known-ambiguous inputs and check how often you get `unknown` versus a confident guess.
- If the sentinel matters and your chosen model won't cooperate, put a confidence gate in front (next section) rather than trusting the prompt.

## When it never stops

A grammar with no path to the end-of-generation token never lets the model stop. `root ::= "a" root` is legal GBNF (it has a `root`, no left recursion) but can only ever produce more `a` — so generation runs until it hits `max_tokens`. This is easy to write by accident with repetition rules. There's no infinite loop (the token cap always wins), but a node that quietly generates its full `max_tokens` every turn is a latency surprise. Give a constrained node a deliberately small `max_tokens` when its grammar's outputs are short.

## Grammar vs tool calling vs intent classification

These three closed-set techniques overlap; pick by what you need out the other side.

| Technique                                                                                          | Gives you                                                                | Reach for it when                                                                                  |
| -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------- |
| **Grammar on `Generate`**                                                                          | Both/all slots in one decode, guaranteed valid; **no** confidence number | You need the *whole* structured answer (`<action> <object>`) and the value set is small/enumerable |
| **[Tool calling](https://docs.tryllengine.com/concepts/tool-calling.html)**                        | Typed, named, possibly-nested arguments as JSON                          | Slots are rich/typed/optional, or many actions each take different arguments                       |
| **[Classify Intent (LLM)](https://docs.tryllengine.com/reference/nodes/classify-intent-llm.html)** | One label **with a calibrated probability** + threshold/margin gates     | You need to *reject* low-confidence input ("was that even a command?"), and you only need the verb |

They compose. The strongest command pipeline for noisy input is `ClassifyIntentLLM` (confidence-gated, so a garbled utterance is *rejected* rather than mapped to a wrong-but-valid command) feeding a grammar-constrained `Generate` for the object slot. Grammar alone is the right, cheap choice when the input is clean or the wrong-but-valid risk is acceptable.

## Mutability: command turn ↔ free chat

`grammar` is **mutable**. The sampler rebuilds it every turn, so a client can flip a single node between a strict "command turn" (grammar set) and ordinary free chat (grammar empty) with [`ChangeAgentParam`](https://docs.tryllengine.com/how-to/change-agent-parameters.html) — no need to recreate the agent. This mirrors how a `ToolCall` node's `mode` is mutable for the [same reason](https://docs.tryllengine.com/concepts/tool-calling.html#forcing-a-tool-call).

## What makes constrained output reliable (or not)

1. **Keep the value sets small and clean-tokenising.** Short, common words beat long or exotic ones; consistent spacing in the grammar matters.
1. **Write the instruction like a spec.** List every allowed word, the slot order, and the sentinel rule. The grammar enforces shape; the prompt drives choice.
1. **Turn thinking off** for the node's model variant.
1. **Pick the model for the hard part** — usually the sentinel/`unknown` discipline — and verify it empirically.
1. **Don't confuse valid with correct.** Add a gate when meaning matters.

## Edges and pitfalls

- **Valid-but-wrong is silent.** There's no error when the model picks the wrong legal value; only your own checks catch it.
- **No confidence from grammar.** A grammar returns text, not a probability. Read a first-token logprob via `ClassifyIntentLLM` if you need gating.
- **`Generate and Speak` + grammar is narrow.** The constrained output is *spoken*, so it only makes sense for a fixed set of barks / canned lines, not for silent command parsing. See its [node reference](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html#constrained-output-gbnf-grammar).
- **Grammar doesn't touch the KV cache.** It affects sampling only; prefix reuse across turns is unchanged.

## Related

- [Reference: Generate node](https://docs.tryllengine.com/reference/nodes/generate.html#constrained-output-gbnf-grammar)
- [Reference: Generate and Speak node](https://docs.tryllengine.com/reference/nodes/generate-and-speak.html#constrained-output-gbnf-grammar)
- [How-to: Constrain output with a grammar](https://docs.tryllengine.com/how-to/constrain-output-with-a-grammar.html)
- [Concept: Tool calling](https://docs.tryllengine.com/concepts/tool-calling.html)
- [Reference: Classify Intent (LLM) node](https://docs.tryllengine.com/reference/nodes/classify-intent-llm.html)
- [How-to: Change agent parameters at runtime](https://docs.tryllengine.com/how-to/change-agent-parameters.html)
