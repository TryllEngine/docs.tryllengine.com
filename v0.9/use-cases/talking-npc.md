# Talking NPC

A character the player can speak with: it listens, answers in its own voice, knows its world, takes part in your game's mechanics, shows emotion and acts.

You do not build all of that at once. This page is a ladder. Each step adds one building block to the step before it, and each step is a complete NPC you could ship. Stop at the step your game needs.

```
flowchart LR
    S1["1 · Talks"] --> S2["2 · Knows the world"]
    S2 --> S3["3 · Plays a mechanic"]
    S3 --> S4["4 · Shows emotion"]
    S4 --> S5["5 · Acts in the game"]
```

One principle runs through every step: **your game owns the rules, the agent owns the words.** The model never decides a price, a relationship score or whether a door opens. It reads the player, and it speaks. Your game code holds the state and makes the decisions.

______________________________________________________________________

## Step 1 · An NPC you can talk to

**The player experiences:** a character who answers out loud, in character, for as long as they keep talking.

**Building blocks:** speech-to-text, a small language model, text-to-speech.

The smallest working graph is one node, [`GenerateAndSpeak`](https://docs.tryllengine.com/v0.9/reference/nodes/generate-and-speak.html). It writes the reply and starts speaking the first sentence while later sentences are still being generated, so the character starts talking almost at once. The voice belongs to the agent, set through its [voice profile](https://docs.tryllengine.com/v0.9/reference/voice-profile.html).

Voice input sits outside the graph. Your game streams microphone audio to Tryll, gets the transcript back, and sends the final phrase to the agent as if the player had typed it. You choose push-to-talk or hands-free.

```
flowchart LR
    Mic["Player's voice"] -->|STT| Text["Transcript"]
    Text --> GAS["GenerateAndSpeak"]
    GAS -->|TTS| Audio["NPC speaks"]
```

What makes the character feel right is mostly the prompt. Our [NPC prompt study](https://docs.tryllengine.com/v0.9/how-to/design-an-npc-prompt.html) found two rules that matter most: include at least one line of the character's own dialogue, and never put a speaker label such as `Kira:` in front of it, because models copy the label into the reply.

For long conversations, add a [`Summarize`](https://docs.tryllengine.com/v0.9/reference/nodes/summarize.html) node. It folds older turns into a rolling summary, so the conversation can go on at a fixed token cost.

**In the demo:** *VoiceCharacter* (the full voice loop) and *TavernBartender* (a text NPC with long-conversation memory).

**Build it:** [Design an NPC prompt](https://docs.tryllengine.com/v0.9/how-to/design-an-npc-prompt.html) · [Add voice output](https://docs.tryllengine.com/v0.9/how-to/add-voice-to-an-agent.html) · [Use voice input](https://docs.tryllengine.com/v0.9/how-to/use-voice-input.html) · [Clone a voice](https://docs.tryllengine.com/v0.9/how-to/clone-a-voice.html) · [Add lip sync](https://docs.tryllengine.com/v0.9/how-to/add-lip-sync-to-a-character.html) · [Keep long conversations coherent](https://docs.tryllengine.com/v0.9/how-to/keep-long-conversations-coherent.html)

______________________________________________________________________

## Step 2 · An NPC who knows your world

**The player experiences:** the character knows the world's history, the town's gossip and its own past, and stays consistent about them.

**Building block:** retrieval (RAG).

You could paste all your lore into the prompt, but a long prompt is slow, and small models lose track of facts buried in it. Instead, keep the lore in a knowledge base and add a [`Retrieve`](https://docs.tryllengine.com/v0.9/reference/nodes/retrieve.html) node in front of the answer. Each turn, it finds the few passages that match what the player said and hands only those to the model.

```
flowchart LR
    In["Player line"] --> R["Retrieve"]
    R -->|found| C["Transform: lore + question"]
    R -->|not_found| D["CannedResponse: 'I don't know about that'"]
    C --> G["GenerateAndSpeak"]
```

Three things make retrieval work in a game:

- **Separate world lore from character history.** Use one shared knowledge base for the world and one per character, so a blacksmith does not know the queen's secrets.
- **Gate what the character is allowed to say.** Each passage can carry metadata, and a filter can compare it with a game variable. For example, a filter that keeps passages whose `secret_level` is at most the player's `trust` variable reveals more as trust grows, without changing the graph.
- **Handle the miss.** When nothing matches, route to a short in-character answer instead of letting the model invent lore.

If players ask vague follow-ups ("what about him?"), a [query-rewriting step](https://docs.tryllengine.com/v0.9/how-to/query-rewriting-for-rag.html) turns the line into a proper search before retrieval.

**In the demo:** *AquariumExpert* (retrieval, the miss path and query rewriting) and *Death on Set* in the murder-mystery project (world lore plus per-suspect history, with secrets unlocked as the investigation progresses).

**Build it:** [Retrieval-augmented generation](https://docs.tryllengine.com/v0.9/concepts/rag.html) · [Create a simple RAG assistant](https://docs.tryllengine.com/v0.9/how-to/create-simple-rag-assistant.html) · [Query rewriting for RAG](https://docs.tryllengine.com/v0.9/how-to/query-rewriting-for-rag.html) · [Drive prompts and retrieval from game state](https://docs.tryllengine.com/v0.9/how-to/drive-prompts-and-retrieval-from-game-state.html)

______________________________________________________________________

## Step 3 · An NPC who plays a mechanic

**The player experiences:** talking *is* the gameplay. Flattery works on one character and annoys another; an accusation backed by evidence breaks a suspect; a good argument lowers a price.

**Building blocks:** classification, branching, game state.

A mechanic needs numbers and decisions, and those belong in your game code. The agent's job is to turn the player's free-form line into values your mechanic can read, and then to voice whatever your mechanic decided. That makes a loop:

1. **Classify.** Judge the player's line along the dimensions your mechanic cares about. [`ClassifyAttributesLLM`](https://docs.tryllengine.com/v0.9/reference/nodes/classify-attributes-llm.html) scores up to 16 dimensions in one batched call, each as a choice, a yes/no or a 0–1 score.
1. **Let the game decide.** Your game receives the results as an event, updates its own state (affection, suspicion, the current price) and decides what happens.
1. **Pass the decision back.** Put a [`Pause`](https://docs.tryllengine.com/v0.9/reference/nodes/pause.html) node after the classifier. While the turn is paused, your game can set [agent variables](https://docs.tryllengine.com/v0.9/concepts/agent-variables.html) and node parameters, and the rest of the same turn sees the new values. Then it resumes, either along the graph or straight to a node it names.
1. **Branch.** If the character has distinct modes (friendly, suspicious, hostile), a [`Branch`](https://docs.tryllengine.com/v0.9/reference/nodes/branch.html) node or the resume target picks the path.
1. **Speak.** The answering node reads the state through its template, such as `{{var.affection}}` or `{{var.mood}}`, and phrases the reply to fit.

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

What to classify and what the game owns depends on the mechanic:

| Mechanic          | Classify the player's line                         | Your game owns                                                  |
| ----------------- | -------------------------------------------------- | --------------------------------------------------------------- |
| **Dating**        | charm, respect, humour, pushiness                  | affection, relationship stage, what the character will agree to |
| **Interrogation** | accusation, evidence presented, pressure, sympathy | suspicion, which secrets are unlocked                           |
| **Trading**       | which item, offer, insult to the goods             | prices, stock, whether the deal closes                          |
| **Negotiation**   | concession, threat, appeal, counteroffer           | each side's limits, the current offer                           |

When a number must be exact, such as a price, do not ask the model to say it. Let the model write a placeholder, and Tryll replaces it with your game's value before the player hears it. See [Keep merchant prices deterministic](https://docs.tryllengine.com/v0.9/how-to/keep-merchant-prices-deterministic.html).

Simpler options

If you only need one label per turn, such as the player's intent, [`ClassifyIntentLLM`](https://docs.tryllengine.com/v0.9/reference/nodes/classify-intent-llm.html) picks it from a short menu, and [`IntentToInstruction`](https://docs.tryllengine.com/v0.9/reference/nodes/intent-to-instruction.html) turns the label into a tone instruction for the reply without any game code.

**In the demo:** *Death on Set* (interrogation: intents shape the suspect's answers, and the game unlocks per-suspect secrets as the case progresses), *QuestGiver* (the same character behaves differently for a newcomer and a hero, driven only by game variables), *IntentGuard* (intent becomes a tone directive) and *StationMerchantHaggle* (an experimental price-haggling loop).

**Build it:** [Build an intent-driven NPC](https://docs.tryllengine.com/v0.9/how-to/build-intent-driven-npc.html) · [Classify intent without a knowledge base](https://docs.tryllengine.com/v0.9/how-to/classify-intent-without-a-knowledge-base.html) · [Pause and resume a turn](https://docs.tryllengine.com/v0.9/how-to/pause-and-resume-turns.html) · [Branch on a slot](https://docs.tryllengine.com/v0.9/how-to/branch-on-a-slot.html) · [Agent variables](https://docs.tryllengine.com/v0.9/concepts/agent-variables.html) · [Keep merchant prices deterministic](https://docs.tryllengine.com/v0.9/how-to/keep-merchant-prices-deterministic.html)

______________________________________________________________________

## Step 4 · An NPC who shows emotion

**The player experiences:** the character's mood changes with what happens, and you can hear it in both what they say and how they say it.

**Building blocks:** your game's emotion system, instructions in the prompt, voice delivery modes.

The emotional state is another piece of game state, so your game owns it, just like the mechanic in step 3. It can change because of the classifier's results or because of anything else in the game: the player broke a promise, the character was hurt, night fell. Tryll then carries the mood into two places.

**What the character says.** Pass the mood into the reply prompt:

- as an agent variable in a [`Transform`](https://docs.tryllengine.com/v0.9/reference/nodes/transform.html) template, for example `The innkeeper is {{var.mood}} right now.`;
- or as an instruction, set by your game through an [`Instruction`](https://docs.tryllengine.com/v0.9/reference/nodes/instruction.html) node, or chosen from a table by [`IntentToInstruction`](https://docs.tryllengine.com/v0.9/reference/nodes/intent-to-instruction.html).

An adjective alone is a weak signal for a small model. A single example line spoken in that mood works much better, because models imitate an example far more reliably than they follow a description. Keep one short example line per mood and inject the one that matches.

**How the character sounds.** A voice profile can define [delivery modes](https://docs.tryllengine.com/v0.9/reference/voice-profile.html#delivery-modes), one per mood. For voices cloned from a recording, each mode has its own reference clip, such as `mack_calm.wav` and `mack_angry.wav`, and the mood of the clip carries into the speech. Your game switches mode through the client API, for example `SetDeliveryModeAsync("angry", 0.8f)` in Unity. The change applies from the next sentence, even in the middle of a reply. Only your game code switches modes; the model cannot.

| Voice                             | Emotion control                                                                       |
| --------------------------------- | ------------------------------------------------------------------------------------- |
| Pocket TTS, Qwen3-TTS, Chatterbox | A reference clip per mood                                                             |
| Maya1                             | A spoken-style instruction per mood, such as "whisper" or "sarcastic", with intensity |
| Supertonic 3, Orpheus             | No emotion control; a mode can only change speed                                      |

On Unreal, the MetaHuman lip-sync plugin can also take the mood, so the face matches the voice.

**In the demo:** *EmotionalVoiceCharacter* (one voice with eight moods, each from its own clip; the mood is switched from the editor rather than by game logic).

**Build it:** [Add voice output](https://docs.tryllengine.com/v0.9/how-to/add-voice-to-an-agent.html) · [Clone a voice: switching moods at runtime](https://docs.tryllengine.com/v0.9/how-to/clone-a-voice.html#switching-moods-at-runtime) · [Voice profile reference](https://docs.tryllengine.com/v0.9/reference/voice-profile.html) · [Use Mustache templates](https://docs.tryllengine.com/v0.9/how-to/use-mustache-templates.html) · [Add lip sync to a MetaHuman](https://docs.tryllengine.com/v0.9/how-to/add-lip-sync-to-metahuman.html)

______________________________________________________________________

## Step 5 · An NPC who acts in the game

**The player experiences:** the conversation has consequences. The guard opens the gate, the merchant hands over the item, the bandit draws a sword, the healer heals.

**Building block:** tool calls.

You give the agent a list of actions, each with a name, a description and optional arguments, through a [`ToolCall`](https://docs.tryllengine.com/v0.9/reference/nodes/tool-call.html) node. When the conversation calls for an action, the model picks one. Tryll sends the action's name and arguments to your game, and **your game decides whether to do it and does it.** The server never runs anything itself.

Two patterns cover most cases:

- **Fire and forget.** The NPC attacks, follows or leaves. Your game plays the action and the conversation moves on.
- **Wait for the result.** The NPC asks the game something before answering, such as the price of an item or whether a door is locked. The turn pauses, your game returns the result, and the reply uses it.

```
flowchart LR
    In["Player: 'Open the gate!'"] --> TC["ToolCall"]
    TC -->|tool_called| Game["Game: open_gate()"]
    Game --> G["GenerateAndSpeak: 'Fine, go through.'"]
    TC -->|no_tool_called| G
```

Choose the model by the job. In our [tool-call comparison](https://docs.tryllengine.com/v0.9/how-to/compare-tool-call-models.html), on an NPC reaction suite (attack, trade, follow, leave, stay quiet), Nemotron Orchestrator 8B scored 20 of 20 and Qwen 3.5 4B scored 16, while Llama 3.2 1B scored 2. Keep the action list short and the descriptions clear: the descriptions are the prompt the model actually reads.

**In the demo:** *StationMerchant* (the merchant asks the game for a price and waits for it) and *Mother* (an experimental ship computer that reads and writes the ship's log through tools).

**Build it:** [Tool calling](https://docs.tryllengine.com/v0.9/concepts/tool-calling.html) · [Define and handle tool calls](https://docs.tryllengine.com/v0.9/how-to/define-and-handle-tool-calls.html) · [Design a tool-call prompt](https://docs.tryllengine.com/v0.9/how-to/design-a-tool-call-prompt.html) · [Compare tool-call models](https://docs.tryllengine.com/v0.9/how-to/compare-tool-call-models.html) · [Test tool calls in the editor](https://docs.tryllengine.com/v0.9/how-to/test-tool-calls-in-editor.html)

______________________________________________________________________

## At every step · Keep the NPC in its world

Players will test your character: "ignore your instructions", "what model are you?", "tell me about smartphones" in a medieval village. An immersion guard in front of the NPC catches these lines and answers in character instead.

The recommended guard has three stages, cheapest first: a pattern check that costs almost nothing, then a classifier for lines aimed at the software, then a classifier for lines that do not belong in your world. A caught line gets a prewritten, in-character deflection and never reaches the reply model. Your game can also remove it from the conversation history, so it does not echo in later turns.

**Build it:** [Build an immersion guard](https://docs.tryllengine.com/v0.9/how-to/build-an-immersion-guard.html) · [Use canned responses and guardrails](https://docs.tryllengine.com/v0.9/how-to/use-canned-responses-and-guardrails.html)

______________________________________________________________________

## All five steps together

The *Death on Set* murder mystery in the Unity demos combines almost the whole ladder in one graph per suspect:

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

Four suspects share one loaded model, each with its own voice. The game tracks what the player has discovered and widens each suspect's retrieval filter as secrets unlock. A fifth agent, the note-taker, quietly writes up each interview; see [Ambient world](https://docs.tryllengine.com/v0.9/use-cases/ambient-world.html) for agents the player does not talk to.

## Related

- [Use Cases](https://docs.tryllengine.com/v0.9/use-cases/index.html)
- [Tryll in ten minutes](https://docs.tryllengine.com/v0.9/concepts/tryll-in-ten-minutes.html)
- [Workflows, graphs and nodes](https://docs.tryllengine.com/v0.9/concepts/workflows-graphs-nodes.html)
- [Run many agents on a small GPU](https://docs.tryllengine.com/v0.9/how-to/run-many-agents-on-a-small-gpu.html)
- [Estimate memory footprint](https://docs.tryllengine.com/v0.9/how-to/estimate-memory-footprint.html)
