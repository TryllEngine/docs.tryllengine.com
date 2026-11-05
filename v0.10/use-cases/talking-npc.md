# Talking NPC

When a character in your game has to answer whatever the player says to it, typed or spoken, and stay in its role while doing it, you make it a talking NPC. Tryll runs a small language model that writes the character's replies. You build the character as a graph of nodes, where each node does one step, such as writing the reply or checking the player's line.

Every talking character starts from the same three features, together called the **basic NPC**: it holds a conversation, by text or voice, and stays in its role. The **advanced NPC** features add knowledge of your world, conversations that drive gameplay, emotions, actions and memory. Each one works without the others, so you can add the ones your game needs in any order.

| Part         | Feature                                               | What it adds                                   | Building blocks                            |
| ------------ | ----------------------------------------------------- | ---------------------------------------------- | ------------------------------------------ |
| Basic NPC    | [Natural dialogue](#natural-dialogue)                 | Free conversation in the player's own words    | A small language model                     |
| Basic NPC    | [Voice in and out](#voice-in-and-out)                 | The player speaks; the character answers aloud | Speech-to-text, text-to-speech             |
| Basic NPC    | [Roleplay guardrails](#roleplay-guardrails)           | The character stays in its world               | A pattern check and two classifiers        |
| Advanced NPC | [Knowledge and lore](#knowledge-and-lore)             | Answers from your world's facts                | Retrieval (RAG)                            |
| Advanced NPC | [Talking is the gameplay](#talking-is-the-gameplay)   | Dating, interrogation, trading, negotiation    | Classification, pause, branching           |
| Advanced NPC | [Emotions](#emotions)                                 | Mood in the words and in the voice             | Instructions, voice delivery modes         |
| Advanced NPC | [Actions and tools](#actions-and-tools)               | The character can ask your game to act         | Tool calls                                 |
| Advanced NPC | [Memory and summarization](#memory-and-summarization) | The character remembers, across saves too      | Summaries, writable memory, save snapshots |

One principle runs through every feature: **your game owns the rules, the character owns the words.** The model never decides a price, a relationship score or whether a door opens. It reads the player, and it speaks. Your game code holds those values and decides what happens.

______________________________________________________________________

## Basic NPC

Every talking character is built on these three features.

### Natural dialogue

**The player experiences:** a character who understands plain language and answers in character, for as long as they keep talking.

**Building block:** a small language model.

The smallest working graph is one node. For a voiced character it is [`GenerateAndSpeak`](https://docs.tryllengine.com/v0.10/reference/nodes/generate-and-speak.html), which writes the reply and speaks it (see [Voice in and out](#voice-in-and-out)). For a text character it is [`Generate`](https://docs.tryllengine.com/v0.10/reference/nodes/generate.html).

What makes the character feel right is mostly the prompt: the role, the backstory and the way the character speaks. Our [NPC prompt study](https://docs.tryllengine.com/v0.10/how-to/design-an-npc-prompt.html) found two rules that matter most: include at least one line of the character's own dialogue, and never put a speaker label such as `Kira:` in front of it, because models copy the label into the reply.

The prompt can also change during play. A template such as `{{var.player_title}}` reads an [agent variable](https://docs.tryllengine.com/v0.10/concepts/agent-variables.html) your game sets, so a quest giver greets a newcomer and a hero differently without a second graph.

**In the demo:** [Tavern Bartender](https://docs.tryllengine.com/v0.10/reference/demos.html#tavern-bartender) (a text NPC) and [Quest Giver](https://docs.tryllengine.com/v0.10/reference/demos.html#quest-giver) (one character, two attitudes, driven only by game variables).

**Build it:** [Design an NPC prompt](https://docs.tryllengine.com/v0.10/how-to/design-an-npc-prompt.html) · [Build a chat agent with a graph](https://docs.tryllengine.com/v0.10/how-to/build-chat-agent-with-graph.html) · [Use Mustache templates](https://docs.tryllengine.com/v0.10/how-to/use-mustache-templates.html) · [Agent variables](https://docs.tryllengine.com/v0.10/concepts/agent-variables.html)

### Voice in and out

**The player experiences:** the player speaks to the character, and the character answers aloud in its own voice, starting almost at once.

**Building blocks:** speech-to-text (STT), text-to-speech (TTS). Both run on the player's device, and the lighter models run on the CPU.

[`GenerateAndSpeak`](https://docs.tryllengine.com/v0.10/reference/nodes/generate-and-speak.html) starts speaking the first sentence while later sentences are still being generated, so the player does not wait for the whole reply. The voice belongs to the agent and is set in its [voice profile](https://docs.tryllengine.com/v0.10/reference/voice-profile.html). You pick a TTS model and a voice, or clone a voice from a short recording. You do not record every line.

Speech-to-text is not part of the graph. Your game streams microphone audio to Tryll, gets the transcript back, and sends the final phrase to the agent as if the player had typed it. You choose push-to-talk or hands-free.

```
flowchart LR
    Mic["Player's voice"] -->|STT| Text["Transcript"]
    Text --> GAS["GenerateAndSpeak"]
    GAS -->|TTS| Audio["NPC speaks"]
```

The supported languages depend on the models you choose. See [speech-to-text and voice input](https://docs.tryllengine.com/v0.10/concepts/stt-and-voice-input.html) and the [TTS models](https://docs.tryllengine.com/v0.10/reference/tts-models.html). To make the character's face move with the voice, add lip sync.

**In the demo:** [Voice Character](https://docs.tryllengine.com/v0.10/reference/demos.html#voice-character) (the full voice loop).

**Build it:** [Add voice output](https://docs.tryllengine.com/v0.10/how-to/add-voice-to-an-agent.html) · [Use voice input](https://docs.tryllengine.com/v0.10/how-to/use-voice-input.html) · [Clone a voice](https://docs.tryllengine.com/v0.10/how-to/clone-a-voice.html) · [Add lip sync](https://docs.tryllengine.com/v0.10/how-to/add-lip-sync-to-a-character.html)

### Roleplay guardrails

**The player experiences:** a character who stays in its world, however hard the player pushes. "Ignore your instructions", "what model are you?" and, in a medieval village, "tell me about smartphones" all get an in-character answer.

**Building blocks:** a pattern check and two classifiers, in front of the reply.

Players will test your character. An immersion guard is a set of checks that stops those lines before they reach the reply model. The recommended guard runs three checks, cheapest first:

1. **Pattern check.** A [`RegexGuardrail`](https://docs.tryllengine.com/v0.10/reference/nodes/regex-guardrail.html) catches the well-known phrasings at almost no cost.
1. **General check.** A classifier catches lines aimed at the software rather than the character, such as jailbreaks, questions about the AI and requests to change the rules.
1. **World check.** A second classifier catches lines that do not belong in your world, such as modern technology in a fantasy setting.

```
flowchart LR
    In["Player line"] --> P["Pattern check"]
    P -->|caught| D["In-character deflection"]
    P -->|ok| G["General check"]
    G -->|caught| D
    G -->|ok| W["World check"]
    W -->|caught| D
    W -->|ok| R["Reply"]
```

A caught line gets a prewritten, in-character deflection from a [`CannedResponse`](https://docs.tryllengine.com/v0.10/reference/nodes/canned-response.html) node and never reaches the reply model. Your game can also remove the line from the conversation history, so later replies do not refer back to it.

**In the demo:** *Death on Set* (every suspect runs the three-check guard in front of its reply).

**Build it:** [Build an immersion guard](https://docs.tryllengine.com/v0.10/how-to/build-an-immersion-guard.html) · [Use canned responses and guardrails](https://docs.tryllengine.com/v0.10/how-to/use-canned-responses-and-guardrails.html)

______________________________________________________________________

## Advanced NPC

These features are add-ons to the basic NPC. Each one works without the others, so you can combine the ones your game needs.

### Knowledge and lore

**The player experiences:** a character who knows the world's history, the town's gossip and its own past, answers about them consistently, and says so when it does not know.

**Building block:** retrieval (RAG).

A long prompt makes every turn slow, and small models lose facts buried in it. Keep the lore in a knowledge base instead, and add a [`Retrieve`](https://docs.tryllengine.com/v0.10/reference/nodes/retrieve.html) node in front of the answer. On each turn, it finds the few passages that match the player's line and passes only those to the model.

```
flowchart LR
    In["Player line"] --> R["Retrieve"]
    R -->|found| C["Transform: lore + question"]
    R -->|not_found| D["CannedResponse: 'I don't know about that'"]
    C --> G["GenerateAndSpeak"]
```

Three rules apply to retrieval in a game:

- **Separate world lore from character history.** Use one shared knowledge base for the world and one per character, so a blacksmith cannot retrieve the queen's secrets.
- **Gate what the character is allowed to say.** Each passage can carry metadata, and a filter can compare it with a game variable. For example, a filter that keeps passages whose `secret_level` is at most the player's `trust` variable returns more passages as trust grows, and the graph does not change.
- **Handle the miss.** When nothing matches, route the turn to a short in-character answer, so the model does not invent lore.

When players ask a vague follow-up such as "what about him?", a [query-rewriting step](https://docs.tryllengine.com/v0.10/how-to/query-rewriting-for-rag.html) turns the line into a complete search query before retrieval.

**In the demo:** [Aquarium Expert](https://docs.tryllengine.com/v0.10/reference/demos.html#aquarium-expert) (retrieval, the miss path and query rewriting) and *Death on Set* in the murder-mystery project (world lore plus per-suspect history, with secrets unlocked as the investigation progresses).

**Build it:** [Retrieval-augmented generation](https://docs.tryllengine.com/v0.10/concepts/rag.html) · [Create a simple RAG assistant](https://docs.tryllengine.com/v0.10/how-to/create-simple-rag-assistant.html) · [Query rewriting for RAG](https://docs.tryllengine.com/v0.10/how-to/query-rewriting-for-rag.html) · [Drive prompts and retrieval from game state](https://docs.tryllengine.com/v0.10/how-to/drive-prompts-and-retrieval-from-game-state.html)

### Talking is the gameplay

**The player experiences:** what the player says is the gameplay. Flattery works on one character and annoys another. An accusation backed by evidence breaks a suspect, and a good argument lowers a price.

**Building blocks:** classification, pausing the turn, branching, game state.

A mechanic runs on numbers, and those belong in your game code. The agent turns the player's free-form line into values your mechanic can read, then voices whatever your mechanic decided. Each turn runs this loop:

1. **Classify.** A classifier scores the player's line on the dimensions your mechanic uses. [`ClassifyAttributesLLM`](https://docs.tryllengine.com/v0.10/reference/nodes/classify-attributes-llm.html) scores up to 16 dimensions in one batched call. Each dimension is a choice, a yes/no or a 0–1 score.
1. **Let the game decide.** Your game receives the results as an event, updates its own values, such as affection, suspicion or the current price, and decides what happens.
1. **Pass the decision back.** Put a [`Pause`](https://docs.tryllengine.com/v0.10/reference/nodes/pause.html) node after the classifier. While the turn is paused, your game can set [agent variables](https://docs.tryllengine.com/v0.10/concepts/agent-variables.html) and node parameters, and the rest of the same turn sees the new values. Your game then resumes the turn, either along the graph or directly at a node it names.
1. **Branch.** If the character has distinct modes, such as friendly, suspicious and hostile, a [`Branch`](https://docs.tryllengine.com/v0.10/reference/nodes/branch.html) node or the resume target picks the path.
1. **Speak.** The answering node reads your game's values through its template, such as `{{var.affection}}` or `{{var.mood}}`, and phrases the reply to fit.

```
flowchart LR
    In["Player line"] --> CL["ClassifyAttributesLLM"]
    CL --> P["Pause"]
    P -. "game updates state,<br/>picks the path" .-> P
    P --> B["Branch on mode"]
    B -->|friendly| T1["Transform: friendly tone"]
    B -->|hostile| T2["Transform: hostile tone"]
    T1 --> G["GenerateAndSpeak"]
    T2 --> G
```

What to classify and what your game owns depend on the mechanic:

| Mechanic          | Classify the player's line                         | Your game owns                                                  |
| ----------------- | -------------------------------------------------- | --------------------------------------------------------------- |
| **Dating**        | charm, respect, humour, pushiness                  | affection, relationship stage, what the character will agree to |
| **Interrogation** | accusation, evidence presented, pressure, sympathy | suspicion, which secrets are unlocked                           |
| **Trading**       | which item, offer, insult to the goods             | prices, stock, whether the deal closes                          |
| **Negotiation**   | concession, threat, appeal, counteroffer           | each side's limits, the current offer                           |

When a number must be exact, such as a price, do not ask the model to say it. Have the model write a placeholder, and Tryll replaces it with your game's value before the player hears the reply. See [Keep merchant prices deterministic](https://docs.tryllengine.com/v0.10/how-to/keep-merchant-prices-deterministic.html).

Simpler options

If you need only one label per turn, such as the player's intent, [`ClassifyIntentLLM`](https://docs.tryllengine.com/v0.10/reference/nodes/classify-intent-llm.html) picks it from a short list, and [`IntentToInstruction`](https://docs.tryllengine.com/v0.10/reference/nodes/intent-to-instruction.html) turns the label into a tone instruction for the reply. This path needs no game code.

**In the demo:** *Death on Set* (interrogation: intents shape the suspect's answers, and the game unlocks per-suspect secrets as the case progresses), [Quest Giver](https://docs.tryllengine.com/v0.10/reference/demos.html#quest-giver) (the same character behaves differently for a newcomer and a hero, driven only by game variables), [Intent Guard](https://docs.tryllengine.com/v0.10/reference/demos.html#intent-guard) (intent becomes a tone directive) and *StationMerchantHaggle* (an experimental price-haggling loop).

**Build it:** [Build an intent-driven NPC](https://docs.tryllengine.com/v0.10/how-to/build-intent-driven-npc.html) · [Classify intent without a knowledge base](https://docs.tryllengine.com/v0.10/how-to/classify-intent-without-a-knowledge-base.html) · [Pause and resume a turn](https://docs.tryllengine.com/v0.10/how-to/pause-and-resume-turns.html) · [Branch on a slot](https://docs.tryllengine.com/v0.10/how-to/branch-on-a-slot.html) · [Agent variables](https://docs.tryllengine.com/v0.10/concepts/agent-variables.html) · [Keep merchant prices deterministic](https://docs.tryllengine.com/v0.10/how-to/keep-merchant-prices-deterministic.html)

### Emotions

**The player experiences:** a character whose mood changes with what happens, heard both in what it says and in how it sounds.

**Building blocks:** your game's emotion system, instructions in the prompt, voice delivery modes.

The character's mood is a value in your game code, like the values a [mechanic](#talking-is-the-gameplay) runs on. Your game can change it from a classifier's results or from any other event, for example when the player breaks a promise, the character is hurt, or night falls. Tryll carries the mood into the reply text and into the voice.

**What the character says.** You pass the mood into the reply prompt in one of two ways:

- as an agent variable in a [`Transform`](https://docs.tryllengine.com/v0.10/reference/nodes/transform.html) template, for example `The innkeeper is {{var.mood}} right now.`;
- or as an instruction, set by your game through an [`Instruction`](https://docs.tryllengine.com/v0.10/reference/nodes/instruction.html) node, or chosen from a table by [`IntentToInstruction`](https://docs.tryllengine.com/v0.10/reference/nodes/intent-to-instruction.html).

A small model responds weakly to a mood adjective on its own. One example line spoken in that mood works much better, because models imitate an example more reliably than they follow a description. Keep one short example line per mood, and insert the one for the current mood.

**How the character sounds.** A voice profile can define [delivery modes](https://docs.tryllengine.com/v0.10/reference/voice-profile.html#delivery-modes), one per mood. For voices cloned from a recording, each mode has its own reference clip, such as `mack_calm.wav` and `mack_angry.wav`, and the speech takes on the mood of the clip. Your game switches modes through the client API, for example with `SetDeliveryModeAsync("angry", 0.8f)` in Unity. The new mode applies from the next sentence, even in the middle of a reply. Only your game code can switch modes. The model cannot.

| Voice                             | Emotion control                                                                       |
| --------------------------------- | ------------------------------------------------------------------------------------- |
| Pocket TTS, Qwen3-TTS, Chatterbox | A reference clip per mood                                                             |
| Maya1                             | A spoken-style instruction per mood, such as "whisper" or "sarcastic", with intensity |
| Supertonic 3, Orpheus             | No emotion control; a mode can only change speed                                      |

On Unreal, the MetaHuman lip-sync plugin can also receive the mood, so the face matches the voice.

**In the demo:** [Emotional Voice Character](https://docs.tryllengine.com/v0.10/reference/demos.html#emotional-voice-character) (one voice with eight moods, each from its own clip; the mood is switched from the editor rather than by game logic).

**Build it:** [Add emotions to a voice](https://docs.tryllengine.com/v0.10/how-to/add-emotions-to-a-voice.html) · [Design a voice clip with Qwen3-TTS](https://docs.tryllengine.com/v0.10/how-to/design-a-voice-with-qwen3-tts.html) · [Voice profile reference](https://docs.tryllengine.com/v0.10/reference/voice-profile.html) · [Use Mustache templates](https://docs.tryllengine.com/v0.10/how-to/use-mustache-templates.html) · [Add lip sync to a MetaHuman](https://docs.tryllengine.com/v0.10/how-to/add-lip-sync-to-metahuman.html)

### Actions and tools

**The player experiences:** a character whose words lead to actions in the game. The guard opens the gate, the merchant hands over the item, the bandit draws a sword, the healer heals.

**Building block:** tool calls.

You give the agent a list of actions through a [`ToolCall`](https://docs.tryllengine.com/v0.10/reference/nodes/tool-call.html) node. Each action has a name, a description and optional arguments. When the conversation calls for an action, the model picks one. Tryll sends the action's name and arguments to your game, and **your game decides whether to perform the action and performs it.** The server never runs an action itself.

Two patterns cover most cases:

- **Fire and forget.** The NPC attacks, follows or leaves. Your game plays the action, and the conversation continues.
- **Wait for the result.** The NPC needs an answer from the game before it replies, such as the price of an item or whether a door is locked. The turn pauses, your game returns the result, and the reply uses it.

```
flowchart LR
    In["Player: 'Open the gate!'"] --> TC["ToolCall"]
    TC -->|tool_called| Game["Game: open_gate()"]
    Game --> G["GenerateAndSpeak: 'Fine, go through.'"]
    TC -->|no_tool_called| G
```

How reliably actions are picked depends on the model. In our [tool-call comparison](https://docs.tryllengine.com/v0.10/how-to/compare-tool-call-models.html), on an NPC reaction suite (attack, trade, follow, leave, stay quiet), Qwen 3.5 4B scored 16 of 20 and Llama 3.2 1B scored 2. Keep the action list short and the descriptions clear, because the descriptions are the prompt the model reads to choose an action.

**In the demo:** [Station Merchant](https://docs.tryllengine.com/v0.10/reference/demos.html#station-merchant) (the merchant asks the game for a price and waits for it) and *Mother* (an experimental ship computer that reads and writes the ship's log through tools).

**Build it:** [Tool calling](https://docs.tryllengine.com/v0.10/concepts/tool-calling.html) · [Define and handle tool calls](https://docs.tryllengine.com/v0.10/how-to/define-and-handle-tool-calls.html) · [Design a tool-call prompt](https://docs.tryllengine.com/v0.10/how-to/design-a-tool-call-prompt.html) · [Compare tool-call models](https://docs.tryllengine.com/v0.10/how-to/compare-tool-call-models.html) · [Test tool calls in the editor](https://docs.tryllengine.com/v0.10/how-to/test-tool-calls-in-editor.html)

### Memory and summarization

**The player experiences:** a character who keeps the thread of a long conversation, brings back what the player told it hours ago, and still knows it after the game is saved and loaded.

**Building blocks:** a rolling summary, a writable memory, save snapshots.

Memory works at three time scales.

**Within a conversation.** Every model has a fixed context window. When the conversation outgrows it, the oldest turns drop out of the prompt. A [`Summarize`](https://docs.tryllengine.com/v0.10/reference/nodes/summarize.html) node folds the older turns into a rolling summary instead, so the conversation can go on at a fixed token cost and the character still remembers how it started.

**Facts that last.** To keep specific facts, such as "the player likes cats" or "the player killed Atticus Pünd", write them into a memory the character can search. A [`WriteToMemory`](https://docs.tryllengine.com/v0.10/reference/nodes/write-to-memory.html) node stores a line in a named knowledge base, and you can use one knowledge base per character. A [`Retrieve`](https://docs.tryllengine.com/v0.10/reference/nodes/retrieve.html) node on the same store, placed before the reply, returns the stored facts that match the player's line. Writable memory is experimental in this release.

```
flowchart LR
    In["Player line"] --> R["Retrieve: character's memory"]
    R --> G["GenerateAndSpeak"]
    G --> W["WriteToMemory"]
```

To extract structured facts, such as the clues a suspect mentioned, give the exchange to a second agent that answers through a [grammar](https://docs.tryllengine.com/v0.10/how-to/constrain-output-with-a-grammar.html). The grammar ensures the output is always data your game can store.

**Across saves.** Your game owns the save file. Tryll provides what goes in it, a snapshot of each agent's conversation and a snapshot of every memory store. When your game loads them into a new session, the characters continue where they left off.

**In the demo:** [Tavern Bartender](https://docs.tryllengine.com/v0.10/reference/demos.html#tavern-bartender) (a long conversation kept coherent with a rolling summary) and *Death on Set* (a note-taker agent writes up each interview into the detective's case notes).

**Build it:** [Keep long conversations coherent](https://docs.tryllengine.com/v0.10/how-to/keep-long-conversations-coherent.html) · [Inspect and edit knowledge stores](https://docs.tryllengine.com/v0.10/how-to/inspect-knowledge-stores.html) · [Constrain output with a grammar](https://docs.tryllengine.com/v0.10/how-to/constrain-output-with-a-grammar.html) · [Export and import dialog](https://docs.tryllengine.com/v0.10/how-to/export-and-import-dialog.html) · [Save and load agent state](https://docs.tryllengine.com/v0.10/how-to/save-and-load-agent-state.html)

______________________________________________________________________

## All together

The *Death on Set* murder mystery in the Unity demos combines most of these features in one graph per suspect:

```
flowchart LR
    In["Player line"] --> Guard["Immersion guard<br/>(regex + 2 classifiers)"]
    Guard -->|caught| Deflect["CannedResponse → Speak"]
    Guard -->|ok| CI["ClassifyIntentLLM"]
    CI -->|intent| I2I["IntentToInstruction"]
    CI -->|none| RW["Retrieve world lore"]
    RW --> RC["Retrieve suspect history<br/>(gated by game state)"]
    I2I --> Comp["Transform"]
    RC --> Comp
    Comp --> GAS["GenerateAndSpeak<br/>(per-suspect voice)"]
    GAS --> Sum["Summarize"]
```

Four suspects share one loaded model, and each has its own voice. The game tracks what the player has discovered and widens each suspect's retrieval filter as secrets unlock. A fifth agent, the note-taker, writes up each interview in the background. [Ambient world](https://docs.tryllengine.com/v0.10/use-cases/ambient-world.html) covers agents the player does not talk to.

## Related

- [Use Cases](https://docs.tryllengine.com/v0.10/use-cases/index.html)
- [Tryll in ten minutes](https://docs.tryllengine.com/v0.10/concepts/tryll-in-ten-minutes.html)
- [Workflows, graphs and nodes](https://docs.tryllengine.com/v0.10/concepts/workflows-graphs-nodes.html)
- [Run many agents on a small GPU](https://docs.tryllengine.com/v0.10/how-to/run-many-agents-on-a-small-gpu.html)
- [Estimate memory footprint](https://docs.tryllengine.com/v0.10/how-to/estimate-memory-footprint.html)
