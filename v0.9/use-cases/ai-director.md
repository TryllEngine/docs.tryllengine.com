# AI Director (Idea)

This is an idea, not a feature

We have not built an AI director, and we have not measured how well small on-device models do this job. This page describes the idea, what published research says about it, and which Tryll building blocks you would use to try it. Every block it needs exists today.

So far, every use case has the model *talking*. An AI director has the model *deciding*: which quest to offer next, how a rival faction responds to the player, what a village does tomorrow, when to raise the tension. Think of a dungeon master who watches the game and plans what happens next.

______________________________________________________________________

## The model plans, the game executes

A language model is good at reading a messy situation and proposing a sensible next move. It is bad at keeping exact rules, counts and world state consistent over a long game. Research on LLM game masters finds exactly this split: models write engaging stories but struggle to run consistent, verifiable game mechanics ([RPGBench](https://arxiv.org/abs/2502.00595)), and they cannot reliably check their own plans ([Kambhampati et al.](https://arxiv.org/abs/2402.01817)).

So divide the work:

- **High level, the model:** pick goals and write a plan. "The bandits raid the mill at dawn, because the player killed their scout and the guard is away."
- **Low level, your game:** carry out the plan with the systems you already have: pathfinding, combat AI, crafting recipes, schedules. These are exact and tested.

The best-known example of this loop is [Voyager](https://arxiv.org/abs/2305.16291): in Minecraft, the model proposes the next task and writes the steps, the game's own API executes them, and the results go back to the model for the next round. [Generative Agents](https://arxiv.org/abs/2304.03442) uses the same split for characters: the model writes each character's plan for the day, and the game moves them around.

```
flowchart LR
    State["Game state<br/>snapshot"] --> Plan["Model writes<br/>a plan (JSON)"]
    Plan --> Check["Game validates<br/>the plan"]
    Check -->|invalid| Plan
    Check -->|valid| Run["Game systems<br/>execute it"]
    Run --> Events["Outcomes<br/>become events"]
    Events --> State
```

______________________________________________________________________

## How you would build it with Tryll

1. **Give it the current state, every time.** Your game sends a short, up-to-date summary of the world with each planning request: who is where, what the player did, what the factions want. Do not rely on the model to remember the world across many turns. Run the director **stateless**, or with a [summary](https://docs.tryllengine.com/v0.9/reference/nodes/summarize.html) at most.
1. **Give it your design.** Put your designers' material in a [knowledge base](https://docs.tryllengine.com/v0.9/concepts/rag.html): quest templates, faction goals, story beats that are allowed to happen. The model then chooses and adapts authored content instead of inventing the world. [Drama Llama](https://arxiv.org/abs/2501.09099) works this way: authors write story fragments with conditions, and the model decides when they fire.
1. **Force a plan your game can read.** Add a grammar to the generation node, so the output is always a JSON plan in your schema. Grammar-constrained decoding produces reliable structured output without fine-tuning ([Geng et al.](https://arxiv.org/abs/2305.13971)). See [Constrained output](https://docs.tryllengine.com/v0.9/concepts/constrained-output.html). Alternatively, let the director call [tools](https://docs.tryllengine.com/v0.9/concepts/tool-calling.html) such as `spawn_patrol(...)` or `offer_quest(...)`; giving an LLM game master game-specific functions improved both its storytelling and its consistency with the game state ([Song et al.](https://arxiv.org/abs/2409.06949)).
1. **Validate before you run.** The grammar guarantees the plan's *shape*, never that it makes sense. Your game checks every plan: does this character exist, is this recipe real, can this patrol reach the mill? If not, send the errors back and ask for a new plan, or fall back to a scripted choice. This "the model proposes, something exact checks" structure is the core recommendation of [Kambhampati et al.](https://arxiv.org/abs/2402.01817), and [LLM+P](https://arxiv.org/abs/2304.11477) takes it further by letting a classical planner do the planning.
1. **Plan in stages.** Ask for the big picture first (this act's three main beats), then the details of each beat in separate calls. A recent world-to-quest pipeline found that each stage conditioned on the structured output of the previous one reduced drift and invented facts ([Borawski et al.](https://arxiv.org/abs/2604.25482)).
1. **Keep it off the player's path.** A director thinks between moments, not during them. Create it with the `Background` workload so its work yields to the characters the player is talking to. See [Ambient world](https://docs.tryllengine.com/v0.9/use-cases/ambient-world.html#keeping-the-world-from-slowing-the-player-down).

An illustrative plan schema, the kind of output a grammar would enforce:

```
{
  "goal": "raid_mill",
  "faction": "red_bandits",
  "reason": "scout killed by player; mill guard reassigned",
  "steps": [
    {"action": "gather", "units": 6, "at": "bandit_camp", "time": "night"},
    {"action": "move",   "to": "old_mill", "time": "dawn"},
    {"action": "attack", "target": "old_mill"}
  ]
}
```

______________________________________________________________________

## Risks to plan for

- **Invalid plans.** Models name places, items and characters that do not exist. Validation in step 4 is not optional.
- **Drift over a long game.** In a benchmark of long interactive stories, the best model tested kept its narrative commitments in only 42% of runs after 20 turns ([NCP-Bench](https://arxiv.org/abs/2608.08160)). Your game, not the model, must hold the canonical state, and send it fresh every time.
- **Latency.** Plan, check and retry loops take several model calls. That is fine for a director that plans the next hour of play, and wrong for anything that must react within a second.
- **Small models are untested here.** The research above uses large cloud models. Whether a 4B to 14B model on the player's GPU plans well enough is an open question. Start small to find out.

## A small first experiment

Do not start with a full story director. Pick one narrow decision with a clear fallback:

- **Choose the next side quest** from ten authored templates and fill in its parameters (giver, target, reward) from the current game state.
- **Plan a villager's day** as a list of activities and places that your schedule system then runs.
- **Pick a faction's response** to the player's last major action from a short menu of authored responses.

Each is one grammar-constrained call, easy to validate and easy to replace with a scripted choice when the plan is rejected.

## Further reading

- Park et al., 2023. [Generative Agents: Interactive Simulacra of Human Behavior](https://arxiv.org/abs/2304.03442)
- Wang et al., 2023. [Voyager: An Open-Ended Embodied Agent with Large Language Models](https://arxiv.org/abs/2305.16291)
- Wang et al., 2023. [Describe, Explain, Plan and Select (DEPS)](https://arxiv.org/abs/2302.01560)
- Liu et al., 2023. [LLM+P: Empowering Large Language Models with Optimal Planning Proficiency](https://arxiv.org/abs/2304.11477)
- Kambhampati et al., 2024. [LLMs Can't Plan, But Can Help Planning in LLM-Modulo Frameworks](https://arxiv.org/abs/2402.01817)
- Song et al., 2024. [Enhancing AI Game Masters with Function Calling](https://arxiv.org/abs/2409.06949)
- Zhu et al., 2023. [CALYPSO: LLMs as Dungeon Masters' Assistants](https://arxiv.org/abs/2308.07540)
- Sun et al., 2025. [Drama Llama: An LLM-Powered Storylets Framework](https://arxiv.org/abs/2501.09099)
- Borawski et al., 2026. [From World-Gen to Quest-Line: A Dependency-Driven Prompt Pipeline](https://arxiv.org/abs/2604.25482)
- Yu et al., 2025. [RPGBench: Evaluating Large Language Models as Role-Playing Game Engines](https://arxiv.org/abs/2502.00595)
- Ma et al., 2026. [Can LLM Agents Stick to the Script? (NCP-Bench)](https://arxiv.org/abs/2608.08160)
- Geng et al., 2023. [Grammar-Constrained Decoding for Structured NLP Tasks without Finetuning](https://arxiv.org/abs/2305.13971)

## Related

- [Use Cases](https://docs.tryllengine.com/v0.9/use-cases/index.html)
- [Constrain output with a grammar](https://docs.tryllengine.com/v0.9/how-to/constrain-output-with-a-grammar.html)
- [Tool calling](https://docs.tryllengine.com/v0.9/concepts/tool-calling.html)
- [Ambient world](https://docs.tryllengine.com/v0.9/use-cases/ambient-world.html)
