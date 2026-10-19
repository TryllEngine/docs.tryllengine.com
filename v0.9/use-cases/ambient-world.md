# Ambient World

Not every character waits for the player to speak. Townsfolk gossip at the market, two guards argue at the gate, a radio host reports last night's raid, a newspaper prints the player's latest scandal. This page covers the agents that **talk on their own**: the player hears or reads them but does not converse with them.

There are two kinds:

- **Background characters**: short lines and overheard conversations that make a place feel alive.
- **Narrators**: a radio, a newspaper, a chronicler or an in-world announcer that turns what happened in the game into a story.

Both follow the same pattern as the [Talking NPC](https://docs.tryllengine.com/v0.9/use-cases/talking-npc.html): **your game owns the events, the agent owns the words.** Your game decides what happened and when it is worth mentioning. The agent decides how to say it.

```
flowchart LR
    Ev["Game events<br/>(raid, trade, rumour)"] --> Game["Your game picks<br/>what to report"]
    Game -->|SendMessage| Agent["Ambient agent"]
    Agent --> Out["Spoken line, radio segment,<br/>newspaper text"]
```

______________________________________________________________________

## Getting game events into the model

The model only knows what you send it. Three ways to pass events, usually combined:

- **Describe the event in the message.** Send a short factual line as the agent's input, such as `The player stole bread from the baker. Two guards saw it.` The agent's prompt tells it to react to the event in character.
- **Put standing facts in variables.** Facts that change slowly, such as the town's mood, the season or the player's reputation, go in [agent variables](https://docs.tryllengine.com/v0.9/concepts/agent-variables.html) and are rendered into the prompt template with `{{var.reputation}}`. Your game updates them; it does not have to repeat them in every message.
- **Seed recent history.** To give an agent a few recent events as context without asking it to answer each one, append them to its history while it is idle. See [Seed and edit dialog history](https://docs.tryllengine.com/v0.9/how-to/seed-and-edit-dialog-history.html).

Decide whether the agent should remember. A gossiping villager who reacts to one event at a time works best **stateless**: every call starts fresh, so old events never leak into new lines. A radio host running a whole evening's show should keep its history, so it can say "as I told you earlier", with a [`Summarize`](https://docs.tryllengine.com/v0.9/reference/nodes/summarize.html) node to keep that history short.

______________________________________________________________________

## Background characters

**Barks.** Short reactive lines ("Hey, watch it!", "Lovely weather today"). Three options, from most to least controlled:

- **Prewritten lines.** A [`CannedResponse`](https://docs.tryllengine.com/v0.9/reference/nodes/canned-response.html) node picks a line from a list, and a [`Speak`](https://docs.tryllengine.com/v0.9/reference/nodes/speak.html) node voices it. No language model is involved.
- **Generated, but from a fixed set.** A grammar on the generation node limits the output to an allowed list, so the model chooses the fitting bark but cannot invent new ones. See [Constrain output with a grammar](https://docs.tryllengine.com/v0.9/how-to/constrain-output-with-a-grammar.html).
- **Free generation.** The model writes a fresh line for the situation. Best for ambient chatter, where variety matters more than control.

**Overheard conversations.** Two background characters talking to each other are two agents, and your game passes the lines between them: A's reply becomes B's input, and so on. Your game controls the whole exchange: who starts, what the topic is (a recent event works well) and when it stops. Cap it at a few lines; long AI-to-AI conversations drift.

```
sequenceDiagram
    participant Game
    participant A as Guard A
    participant B as Guard B
    Game->>A: "Topic: the player was seen near the vault."
    A-->>Game: line 1 (spoken)
    Game->>B: line 1
    B-->>Game: line 2 (spoken)
    Game->>A: line 2
    A-->>Game: line 3 (spoken), game ends the exchange
```

Each speaking character has its own [voice profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html), so the guards do not sound alike.

______________________________________________________________________

## Narrators: radio, newspaper, chronicler

A narrator is one agent with a strong style and a stream of events.

1. **Collect events.** Your game logs what happens: quests finished, fights won, prices rising, the player's notable choices.
1. **Pick and batch.** At the right moment (the radio's next segment, the morning edition, the end of a chapter) your game selects the events worth reporting and sends them to the narrator in one message.
1. **Write in style.** The narrator's prompt sets the voice: a tabloid headline writer, a breathless sports commentator, a dry royal chronicler. One example paragraph in that style steers a small model better than any description.
1. **Deliver.** For a newspaper or a journal, display the text. For a radio or an announcer, use [`GenerateAndSpeak`](https://docs.tryllengine.com/v0.9/reference/nodes/generate-and-speak.html) with a narrator voice.

Keep the facts in your game's hands. If the newspaper must state the exact bounty or the score, pass it as a variable and let Tryll insert it into the text, as described in [Substitute agent variables in output](https://docs.tryllengine.com/v0.9/how-to/substitute-agent-variables-in-output.html). The model then cannot get the number wrong.

A narrator can also sit inside a conversation. One turn can produce a narrator line and a character line, each tagged with the node that wrote it, so your UI can show them differently. See [Send multiple answers in one turn](https://docs.tryllengine.com/v0.9/how-to/send-multiple-answers-in-a-turn.html).

______________________________________________________________________

## Keeping the world from slowing the player down

Ambient agents compete with the conversation the player is actually having, and with your game's rendering, for the same GPU. Tryll has two controls for this.

- **Background priority.** Create ambient agents with the `Background` workload. Nobody is waiting on their output, so the server paces them and gives way to the agent the player is talking to. The workload is a scheduling hint only: it changes when the work runs, never what is generated.
- **Inference throttle.** When your game needs the GPU back, for example during a heavy fight or a level load, it can tell the server to yield. Background work slows first; visible streaming text slows last. See [Inference throttle and agent workload](https://docs.tryllengine.com/v0.9/reference/wire-protocol.html#inference-throttle-and-agent-workload).

```
// Set before the agent is created; it is fixed for the agent's lifetime.
agentComponent.Workload = TryllWorkload.Background;
```

```
from tryll_client import AgentWorkload

narrator = client.create_agent(graph, workload=AgentWorkload.Background)
```

Ambient agents are also cheap in memory when they share a model. A model loads once and every agent that uses it shares the weights; each agent adds only its own working cache. Give ambient agents small context sizes, since their prompts are short. See [Run many agents on a small GPU](https://docs.tryllengine.com/v0.9/how-to/run-many-agents-on-a-small-gpu.html).

______________________________________________________________________

## Ideas

| Idea           | Kind       | Input from the game                     | Output                                |
| -------------- | ---------- | --------------------------------------- | ------------------------------------- |
| Market gossip  | Background | Recent player actions, town mood        | Overheard lines between two villagers |
| Guard barks    | Background | Player is sneaking, running, armed      | One spoken line from a fixed set      |
| War radio      | Narrator   | Battles won and lost, territory changes | Spoken news segment                   |
| Town newspaper | Narrator   | The player's notable deeds and scandals | Headline and short article            |
| Captain's log  | Narrator   | Mission events                          | Written journal entry                 |
| Case notes     | Narrator   | Each finished interrogation             | Running investigation summary         |

**In the demo:** the *Death on Set* murder mystery has a note-taker: after every interview, the game sends a summary of the exchange and the note-taker updates the detective's case notes. The other ideas have no demo yet.

## Related

- [Use Cases](https://docs.tryllengine.com/v0.9/use-cases/index.html)
- [Talking NPC](https://docs.tryllengine.com/v0.9/use-cases/talking-npc.html)
- [Agents and sessions](https://docs.tryllengine.com/v0.9/concepts/agents-and-sessions.html)
- [Add voice output](https://docs.tryllengine.com/v0.9/how-to/add-voice-to-an-agent.html)
- [Estimate memory footprint](https://docs.tryllengine.com/v0.9/how-to/estimate-memory-footprint.html)
