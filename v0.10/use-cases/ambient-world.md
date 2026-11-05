# Ambient World

Parts of your world can talk without waiting for the player: townsfolk gossiping at the market, two guards arguing at the gate, a radio host reporting last night's raid, a newspaper printing the player's latest scandal. In Tryll you build these as ambient agents, which **talk on their own**. The player hears or reads them but does not converse with them.

There are two kinds:

- **Background characters**: short lines and overheard conversations that fill a place with everyday talk.
- **Narrators**: a radio, a newspaper, a chronicler or an in-world announcer that turns what happened in the game into a story.

Both follow the same principle as the [Talking NPC](https://docs.tryllengine.com/v0.10/use-cases/talking-npc.html): **your game owns the events, the agent owns the words.** Your game decides what happened and when it is worth mentioning. The agent writes how it is said.

```
flowchart LR
    Ev["Game events<br/>(raid, trade, rumour)"] --> Game["Your game picks<br/>what to report"]
    Game -->|SendMessage| Agent["Ambient agent"]
    Agent --> Out["Spoken line, radio segment,<br/>newspaper text"]
```

______________________________________________________________________

## Getting game events into the model

The model only knows what you send it. There are three ways to pass events, and games usually combine them:

- **Describe the event in the message.** Send a short factual line as the agent's input, such as `The player stole bread from the baker. Two guards saw it.` The agent's prompt tells it to react to the event in character.
- **Put standing facts in variables.** Facts that change slowly, such as the town's mood, the season or the player's reputation, go in [agent variables](https://docs.tryllengine.com/v0.10/concepts/agent-variables.html) and are rendered into the prompt template with `{{var.reputation}}`. Your game updates them and does not repeat them in every message.
- **Seed recent history.** To give an agent recent events as context without making it answer each one, append them to its history while it is idle. See [Seed and edit dialog history](https://docs.tryllengine.com/v0.10/how-to/seed-and-edit-dialog-history.html).

You decide whether each agent remembers. A gossiping villager who reacts to one event at a time works best **stateless**: every call starts fresh, so old events never leak into new lines. A radio host running a whole evening's show should keep its history so it can say "as I told you earlier". A [`Summarize`](https://docs.tryllengine.com/v0.10/reference/nodes/summarize.html) node keeps that history short.

______________________________________________________________________

## Background characters

**Barks.** Short reactive lines ("Hey, watch it!", "Lovely weather today"). Three options, from most to least controlled:

- **Prewritten lines.** A [`CannedResponse`](https://docs.tryllengine.com/v0.10/reference/nodes/canned-response.html) node picks a line from a list, and a [`Speak`](https://docs.tryllengine.com/v0.10/reference/nodes/speak.html) node voices it. No language model is involved.
- **Generated, but from a fixed set.** A grammar on the generation node limits the output to an allowed list, so the model chooses the fitting bark but cannot invent new ones. See [Constrain output with a grammar](https://docs.tryllengine.com/v0.10/how-to/constrain-output-with-a-grammar.html).
- **Free generation.** The model writes a fresh line for the situation. This suits ambient chatter, where variety matters more than control.

**Overheard conversations.** Two background characters talking to each other are two agents. Your game passes the lines between them, so A's reply becomes B's input, and so on. Your game also decides who starts, what the topic is (a recent event works well) and when the exchange stops. Cap it at a few lines, because long conversations between two models drift.

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

Each speaking character has its own [voice profile](https://docs.tryllengine.com/v0.10/reference/voice-profile.html), so the guards do not sound alike.

______________________________________________________________________

## Narrators: radio, newspaper, chronicler

A narrator is one agent that writes about a stream of game events in a fixed style.

1. **Collect events.** Your game logs what happens: quests finished, fights won, prices rising, the player's notable choices.
1. **Pick and batch.** At the right moment (the radio's next segment, the morning edition, the end of a chapter) your game selects the events worth reporting and sends them to the narrator in one message.
1. **Write in style.** The narrator's prompt sets the voice: a tabloid headline writer, a breathless sports commentator, a dry royal chronicler. One example paragraph in that style steers a small model better than any description of it.
1. **Deliver.** For a newspaper or a journal, display the text. For a radio or an announcer, use [`GenerateAndSpeak`](https://docs.tryllengine.com/v0.10/reference/nodes/generate-and-speak.html) with a narrator voice.

Exact figures should come from your game, not the model. If the newspaper must state the exact bounty or the score, pass it as a variable and Tryll inserts it into the text, as described in [Substitute agent variables in output](https://docs.tryllengine.com/v0.10/how-to/substitute-agent-variables-in-output.html). The model then cannot get the number wrong.

A narrator can also sit inside a conversation. One turn can produce a narrator line and a character line, each tagged with the node that wrote it, so your UI can show them differently. See [Send multiple answers in one turn](https://docs.tryllengine.com/v0.10/how-to/send-multiple-answers-in-a-turn.html).

______________________________________________________________________

## Keeping the world from slowing the player down

Ambient agents share the GPU with the conversation the player is having and with your game's rendering. Tryll has two controls for this.

- **Background priority.** Create ambient agents with the `Background` workload. Nobody is waiting on their output, so the server paces them and gives way to the agent the player is talking to. The workload is only a scheduling hint. It changes when the work runs, never what is generated.
- **Inference throttle.** When your game needs the GPU back, for example during a heavy fight or a level load, it can tell the server to yield. Background work slows first; visible streaming text slows last. See [Inference throttle and agent workload](https://docs.tryllengine.com/v0.10/reference/wire-protocol.html#inference-throttle-and-agent-workload).

```
// Set before the agent is created; it is fixed for the agent's lifetime.
agentComponent.Workload = TryllWorkload.Background;
```

On the `UTryllAgentComponent`, set **Workload** to **Background** in the Details panel, or from C++ before the agent is created:

```
// Set before the agent is created; it is fixed for the agent's lifetime.
AgentComponent->Workload = ETryllWorkload::Background;
```

```
Tryll::Client::AgentCreateOptions opts;
opts.workload = Tryll::AgentWorkload_Background;
auto narrator = client.CreateAgent(graph, std::move(opts));
```

```
from tryll_client import AgentWorkload

narrator = client.create_agent(graph, workload=AgentWorkload.Background)
```

Ambient agents that share a model also cost little memory. The model loads once and every agent that uses it shares the weights. Each agent adds only its own working cache. Give ambient agents small context sizes, since their prompts are short. See [Run many agents on a small GPU](https://docs.tryllengine.com/v0.10/how-to/run-many-agents-on-a-small-gpu.html).

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

**In the demo:** the *Death on Set* murder mystery has a note-taker agent. After every interview, the game sends it a summary of the exchange and it updates the detective's case notes. The other ideas have no demo yet.

## Related

- [Use Cases](https://docs.tryllengine.com/v0.10/use-cases/index.html)
- [Talking NPC](https://docs.tryllengine.com/v0.10/use-cases/talking-npc.html)
- [Agents and sessions](https://docs.tryllengine.com/v0.10/concepts/agents-and-sessions.html)
- [Add voice output](https://docs.tryllengine.com/v0.10/how-to/add-voice-to-an-agent.html)
- [Estimate memory footprint](https://docs.tryllengine.com/v0.10/how-to/estimate-memory-footprint.html)
