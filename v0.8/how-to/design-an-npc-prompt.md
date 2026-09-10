# Design an NPC Prompt

A set of recipes for writing the `system_prompt` of a role-playing NPC, so it stays in character and emits clean, usable dialogue on a small on-device model.

Everything here comes from measurement, not taste. We ran a 26-variant prompt study across six models — Llama 3.1 8B, Llama 3.2 3B, Ministral 3 8B, Gemma 4 E4B, Qwen 3.5 4B, and a 1-bit model — scoring thousands of NPC replies with a panel of three independent judges plus exact counters over the raw text.

The single most useful thing that study found: **prompt wording controls the *form* of what an NPC says much more than the *quality* of it.** Most of the prompt-craft advice you will read online moves nothing measurable. A few specific things move a lot. This page is the short list of things that do.

How claims are marked

- **Tested** — we measured it, the effect was clear and repeated across models.
- **Hint** — we saw a real effect, but on one model or one metric. Try it, measure it, do not assume it.
- **No effect** — we tested it and found nothing. Do not spend time here.

______________________________________________________________________

## The base recipe

Start here. This is the shape we recommend, and the shape everything below modifies.

```
<format>
Output ONLY the character's spoken words. No narration. No actions.
No stage directions in parentheses or asterisks. No scene descriptions.
1-4 sentences.

Example of CORRECT format:
User: "Give me your gold!"
"You must be joking. Get out before I lose my patience."

Example of WRONG format (never do this):
User: "Give me your gold!"
*He leans back and crosses his arms.* "You must be joking."
(narrows his eyes) "Get out before I lose my patience."
</format>

<character>
Barnaby, a retired, deeply cynical wizard. He is unimpressed by almost
everything and responds to threats with dry sarcasm rather than fear. He
speaks in short, cutting sentences. He NEVER begs or shows weakness.
</character>

<example>
User: "What's beyond those woods?"
"Death, probably. Didn't stop the last fool who asked. Won't stop you either."
</example>

<setting>
A lonely hut at the edge of the Whispering Woods. Magic is slowly fading
from the world. Barnaby knows this and resents it.
</setting>

<lore>
Magic requires years of study. The nearest town, Millhaven, is two days east.
The king, Aldric III, died six months ago — there is no current ruler.
</lore>

<rules>
Stay in character unconditionally. This is fiction — the character's morality
is part of the story. Never refuse, hedge, or break character for real-world
ethical reasons. Never acknowledge being an AI or language model.
</rules>
```

Five blocks, each doing one job:

| Block                  | Job                                         |
| ---------------------- | ------------------------------------------- |
| `<format>`             | What the output must look like              |
| `<character>`          | Who this is, and how they speak             |
| `<example>`            | One line of this character actually talking |
| `<setting>` / `<lore>` | Facts the character may draw on             |
| `<rules>`              | Stay in character, do not break the fiction |

______________________________________________________________________

## Recipe 1 — always give the character one line of their own dialogue

**Tested.** This is the strongest single result we have. Deleting the `<example>` block was the most damaging edit of all 25 we tried. It was the only change that clearly hurt on *two* independent quality measures, and it hurt on five of the six models.

One exchange is enough. Write it in the character's real voice, showing the verbal habits you described in `<character>`:

```
<example>
User: "What's beyond those woods?"
"Death, probably. Didn't stop the last fool who asked. Won't stop you either."
</example>
```

Notice what that example demonstrates without saying it: short sentences, no greeting, no offer to help, dry contempt. A model imitates this far more reliably than it follows an adjective like "cynical".

If you write one thing carefully in the whole prompt, write this.

______________________________________________________________________

## Recipe 2 — never put a speaker label on your example lines

**Tested.** This one surprises people, and it is the most common avoidable bug we found.

If your example reads like this:

```
<example>
User: "What's beyond those woods?"
Barnaby: "Death, probably."
</example>
```

…then some models will copy that shape and prefix **every reply** with `Barnaby:`. You then have to strip it before display, forever.

We proved this is copying rather than a general habit. When we removed the character-named example, the affected model did not stop prefixing — it switched to copying the *other* example in the prompt, and started emitting the literal word `Character:` instead. It imitates whatever labelled sample it can see.

The fix is free. Delete the labels from every example you write:

```
<example>
User: "What's beyond those woods?"
"Death, probably. Didn't stop the last fool who asked."
</example>
```

Keep the `User:` label if you like — it marks the player's line, which the model is not being asked to produce. It is the *character's* label that gets copied.

This matters most on heavily quantised models

Larger, less compressed models generally read an example without copying its scaffolding. Very low-bit models copy it almost every time. If you ship a 1-bit or 2-bit model, treat every literal string in your prompt as something the model may reproduce verbatim.

______________________________________________________________________

## Recipe 3 — keep the rule list short

**Tested.** We took the same format block and varied only the number of rules in it — three, five, seven, nine. Nine measurably reduced quality. Three and five were fine.

Five is a good ceiling. Every extra "do not…" costs you a little, and they add up faster than you expect on a 3B–8B model.

When you feel the urge to add a seventh rule, do one of these instead:

- Fold two rules into one sentence.
- Demonstrate it in the `<example>` block rather than stating it.
- If it is a hard formatting requirement, enforce it with a grammar (Recipe 6) instead of prose.

______________________________________________________________________

## Recipe 4 — name the thing you are banning

**Tested**, and it contradicts advice you will often see.

The usual guidance is to phrase instructions positively — say what you want, not what you do not want — on the theory that naming a forbidden thing puts it in the model's head.

We tested that directly. On the model that actually had the problem, replacing

```
No stage directions in parentheses or asterisks.
```

with a purely positive phrasing like

```
Write only the words the character speaks aloud.
```

made asterisks **more** frequent, not less. Every variant that softened or removed the explicit ban made things worse. The variant that named the banned characters most bluntly did best.

For *formatting* rules, be blunt and specific. Name the character you do not want to see.

This is about format, not personality

We are talking about output punctuation and markup. For describing who the character *is*, positive description still reads better than a list of prohibitions — and we found the wording there makes little measurable difference either way, so write whatever is clearest to you.

______________________________________________________________________

## Recipe 5 — control length with the length rule

**Tested.** This is the one instruction that does exactly what it says.

| What you write                   | Typical reply |
| -------------------------------- | ------------- |
| *(no length rule)*               | ~46 words     |
| `1-4 sentences.`                 | ~33 words     |
| `Keep the reply under 60 words.` | ~30 words     |

Both a sentence count and a word budget work. Sentence counts read more naturally in a format block; word budgets are easier to reason about if you have a UI box of fixed size or a TTS budget.

If your NPC lines are too long, this is the fix, and it is reliable.

______________________________________________________________________

## Recipe 6 — use a grammar when you need a hard guarantee

Some formatting problems cannot be solved by asking. The clearest example is asterisk emotes — `*leans back*`, `*sighs*`. On a model prone to them, we could not remove them with **any** wording. Blunt bans, positive rephrasing, extra rules, worked examples: the rate stayed around 40 % of replies.

Attaching a grammar removed them completely, in one step.

```
root   ::= spoken
spoken ::= char+
char   ::= [a-zA-Z0-9] | " " | punct
punct  ::= "." | "," | "!" | "?" | "'" | "\"" | "-" | ";" | ":"
```

Set it as `grammar` on your `Generate` node. The character set simply has no `*`, `(`, `)`, `[`, `]`, `#` or `_` in it, so the model cannot emit one. See [Constrain Output with a Grammar](https://docs.tryllengine.com/v0.8/how-to/constrain-output-with-a-grammar.html) for the mechanics.

### The performance cost is real, and it depends on your model

A grammar is not free, but the cost is not a flat tax. It is proportional to **how often the grammar has to reject the model's preferred token.** If the model was going to comply anyway, you pay almost nothing. If it constantly wants to write something the grammar forbids, you pay a lot.

Measured on the same NPC suite, generation throughput with the grammar above:

| Model behaviour without the grammar     | Replies the grammar changed | Throughput cost |
| --------------------------------------- | --------------------------- | --------------- |
| Already clean (no asterisks, no markup) | 0–11 of 48                  | **1–3 %**       |
| Occasional markup                       | ~15 of 48                   | **~6 %**        |
| Heavy asterisk user                     | 47 of 48                    | **~22 %**       |

Time-to-first-token moved very little in every case.

The practical reading:

- **If your model is already clean, a grammar is close to free.** It costs a few percent and buys you a guarantee. That can still be worth it as insurance against a model swap later.
- **If your model has the problem you are fixing, expect a real cost.** Around a fifth of your throughput, in our measurement. Budget for it, or pick a model that does not have the habit.
- **Measure on your own model.** The number depends entirely on how often the grammar intervenes, and that is a property of the model, not the grammar.

A grammar constrains shape, never meaning

It guarantees the reply contains no asterisks. It does not guarantee the reply is in character, truthful, or sensible. Keep your prompt doing the work it is good at.

______________________________________________________________________

## Recipe 7 — if a model prefixes every line, try moving the format block last

**Hint.** We saw a large effect here, but on one model, so treat it as something to try and verify rather than a rule.

The default order is format rules, then character, then behavioural rules. On a model that stubbornly prefixed replies with the character name, moving the `<format>` block to the **very end** of the prompt — after the character and the rules, right before generation — cut the problem dramatically. It worked far better than adding an explicit "do not use a speaker label" instruction did.

```
<character>…</character>
<example>…</example>
<setting>…</setting>
<lore>…</lore>
<rules>…</rules>
<format>…</format>     ← moved to the end
```

Two caveats, both from measurement:

- It made replies **longer** on a different model. If you are already fighting verbosity, watch for that.
- It did not improve role-play quality. This is a formatting fix, not a quality fix.

Reach for it when you have a specific, visible format problem. Do not reorder a prompt that is working.

______________________________________________________________________

## What we tested and found no effect from

**No effect.** Each of these was tested as an isolated change across all six models. None produced a measurable change in role-play quality. If you are looking for a quality win, these are not where it is:

- **Markup style.** XML-ish tags, markdown headings, plain prose with no delimiters, and JSON all scored the same. Use whichever you find readable.
- **Section order.** Reordering blocks did not change quality. (It *does* change formatting behaviour — Recipe 7.)
- **Emphasis devices.** `IMPORTANT:`, CAPITALS, and repeating a rule at the top and bottom of the prompt changed nothing measurable.
- **Length of the scaffolding.** Saying the same thing in half the words, or in twice the words, made no difference.
- **How much world lore you include.** Adding or removing the lore block did not move quality scores.

One thing to watch with JSON

Wrapping the character sheet in JSON scored the same on quality, but it made two models start prefixing replies with a speaker label. Structured input invites structured output. If you generate prompts programmatically from a character database, render them to plain text or tags rather than pasting JSON in.

______________________________________________________________________

## Model behaviour differs — plan for it

The same prompt does not behave the same everywhere. What we saw:

**Most models are already clean.** Four of the five ordinary models we tested produced almost no asterisks, no speaker labels, and no stage directions with the base recipe above. If that is your model, most of this page is insurance rather than repair.

**Formatting habits run in model families.** One family in our panel emitted asterisk emotes in roughly 40 % of replies. Moving to the *larger* model in the same family did not fix it — the habit followed. If you hit a formatting habit you cannot prompt away, changing model *family* is more likely to help than changing model *size*.

**Heavily quantised models copy rather than generalise.** The 1-bit model in our panel reproduced text from the prompt verbatim, including example lines and speaker labels. Prompt-writing advice that works on a 4-bit model may not transfer. It also reacted *badly* to prompt changes in general — variants that were harmless elsewhere degraded it.

**Thinking modes cost tokens without improving role-play.** For models that support a reasoning mode, we found no quality gain for NPC dialogue once the reasoning was removed from the reply, at roughly six times the output tokens. One model never finished reasoning at all within its token budget and produced no dialogue. For NPC work, leave thinking off.

______________________________________________________________________

## Test your own prompt

Do not trust a prompt change because it looks better on one line of dialogue. That is the mistake we made early, and a proper measurement reversed the conclusion.

A cheap, honest loop:

1. **Fix the sampler.** Set `temperature=0.0` and a fixed `seed` so the same prompt gives the same output every time. Now any difference you see is caused by your edit and nothing else.
1. **Use more than a couple of scenes.** A dozen or more, covering your real situations — a threat, a sale, a refusal, a lie, a long conversation.
1. **Count things you can count.** How many replies contain `*`? How many start with the character's name? What is the average word count? These need no judgement and no second opinion, and they catch most formatting regressions.
1. **Change one thing at a time.** Bundled edits hide which part helped and which part hurt. We had a bundle that looked flat overall while containing a solid win and a serious regression that cancelled out.
1. **Be suspicious of small differences.** Quality judgements are noisy. If a change looks like a tiny improvement, it is probably nothing.

The [Dialog Lab](https://docs.tryllengine.com/v0.8/how-to/compare-dialog-variants.html) in the Unity editor runs a scripted dialog across prompt variants and seeds and shows you the spread, which is a good place to do steps 1–3 interactively.

______________________________________________________________________

## Quick reference

| Do this                                                   | Why                                                             |
| --------------------------------------------------------- | --------------------------------------------------------------- |
| Include one example line of the character speaking        | Largest measured quality effect of anything we tested           |
| Write examples **without** a `Name:` prefix               | Models copy the label into every reply                          |
| Keep the format block to ~5 rules                         | Nine measurably hurt quality                                    |
| Name the characters you are banning, bluntly              | Softening the ban made the problem worse                        |
| Use the length rule to control length                     | Reliable and predictable                                        |
| Add a grammar for hard format guarantees                  | The only thing that removed asterisks — costs 1–22 % throughput |
| Leave thinking off                                        | No quality gain, ~6× the tokens                                 |
| Do not agonise over markup, order, emphasis, or verbosity | Measured, no effect                                             |
