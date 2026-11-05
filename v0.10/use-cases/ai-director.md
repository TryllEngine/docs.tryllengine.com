# AI Director (Idea)

Many games need a system that decides what happens next: which quest to offer, how a rival faction responds to the player, what a village does tomorrow, when to raise the tension. An AI director gives that job to a language model. Like a dungeon master, the director watches the game and plans the next events, and your game carries them out.

This is an idea, not a feature

We have not built an AI director, and we have not measured how well small on-device models do this job. Every Tryll building block it needs exists today.

______________________________________________________________________

## The model plans, the game executes

A language model is good at proposing a sensible next move from a messy situation. It is bad at keeping exact rules, counts and world state consistent over a long game. Research on LLM game masters finds the same split. Models write engaging stories but struggle to run consistent, verifiable game mechanics ([RPGBench](https://arxiv.org/abs/2502.00595)), and they cannot reliably check their own plans ([Kambhampati et al.](https://arxiv.org/abs/2402.01817)).

Divide the work this way:

- **High level, the model:** picks goals and writes a plan, for example "The bandits raid the mill at dawn, because the player killed their scout and the guard is away."
- **Low level, your game:** carries out the plan with the systems it already has, such as pathfinding, combat AI, crafting recipes and schedules. These systems are exact and tested.

The best-known example of this loop is [Voyager](https://arxiv.org/abs/2305.16291). In Minecraft, the model proposes the next task and writes the steps, the game's own API executes them, and the results go back to the model for the next round. [Generative Agents](https://arxiv.org/abs/2304.03442) uses the same split for characters. The model writes each character's plan for the day, and the game moves the characters.

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

1. **Give it the current state, every time.** With each planning request, your game sends a short, up-to-date summary of the world: who is where, what the player did, what the factions want. Do not rely on the model to remember the world across many turns. Run the director **stateless**, or with a [summary](https://docs.tryllengine.com/v0.10/reference/nodes/summarize.html) at most.
1. **Give it your design.** Put your designers' material, such as quest templates, faction goals and the story beats that are allowed to happen, in a [knowledge base](https://docs.tryllengine.com/v0.10/concepts/rag.html). The model then chooses and adapts authored content instead of inventing the world. [Drama Llama](https://arxiv.org/abs/2501.09099) works this way. Authors write story fragments with conditions, and the model decides when they fire.
1. **Force a plan your game can read.** Add a grammar to the generation node so the output is always a JSON plan in your schema. Grammar-constrained decoding produces reliable structured output without fine-tuning ([Geng et al.](https://arxiv.org/abs/2305.13971)). See [Constrained output](https://docs.tryllengine.com/v0.10/concepts/constrained-output.html). The director can instead call [tools](https://docs.tryllengine.com/v0.10/concepts/tool-calling.html) such as `spawn_patrol(...)` or `offer_quest(...)`. Giving an LLM game master game-specific functions improved both its storytelling and its consistency with the game state ([Song et al.](https://arxiv.org/abs/2409.06949)).
1. **Validate before you run.** The grammar guarantees the plan's *shape*, not that the plan makes sense. Your game checks every plan: that each character exists, that each recipe is real, that the patrol can reach the mill. If a check fails, send the errors back and ask for a new plan, or fall back to a scripted choice. Letting the model propose and something exact check is the core recommendation of [Kambhampati et al.](https://arxiv.org/abs/2402.01817). [LLM+P](https://arxiv.org/abs/2304.11477) goes further and lets a classical planner do the planning.
1. **Plan in stages.** Ask for the big picture first, such as this act's three main beats, then for the details of each beat in separate calls. In a recent world-to-quest pipeline, conditioning each stage on the structured output of the previous one reduced drift and invented facts ([Borawski et al.](https://arxiv.org/abs/2604.25482)).
1. **Keep it off the player's path.** A director plans between moments of play. Create it with the `Background` workload so its work yields to the characters the player is talking to. See [Ambient world](https://docs.tryllengine.com/v0.10/use-cases/ambient-world.html#keeping-the-world-from-slowing-the-player-down).

For example, a grammar could enforce a plan like this:

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
- **Drift over a long game.** In a benchmark of long interactive stories, the best model tested kept its narrative commitments in only 42% of runs after 20 turns ([NCP-Bench](https://arxiv.org/abs/2608.08160)). Your game, not the model, must hold the canonical state and send it fresh every time.
- **Latency.** A plan, check and retry loop takes several model calls. That is acceptable for a director that plans the next hour of play, and too slow for anything that must react within a second.
- **Small models are untested here.** The research above uses large cloud models. Whether a 4B to 14B model on the player's GPU plans well enough is an open question. Start small to find out.

## A small first experiment

Do not start with a full story director. Pick one narrow decision with a clear fallback:

- **Choose the next side quest** from ten authored templates and fill in its parameters (giver, target, reward) from the current game state.
- **Plan a villager's day** as a list of activities and places that your schedule system then runs.
- **Pick a faction's response** to the player's last major action from a short menu of authored responses.

Each is one grammar-constrained call. Its plan is easy to validate, and easy to replace with a scripted choice when rejected.

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

- [Use Cases](https://docs.tryllengine.com/v0.10/use-cases/index.html)
- [Constrain output with a grammar](https://docs.tryllengine.com/v0.10/how-to/constrain-output-with-a-grammar.html)
- [Tool calling](https://docs.tryllengine.com/v0.10/concepts/tool-calling.html)
- [Ambient world](https://docs.tryllengine.com/v0.10/use-cases/ambient-world.html)
