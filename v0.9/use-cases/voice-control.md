# Voice Control

The player speaks, and the game does it. Order a squad to flank, cast a spell by saying its name, tell a companion to wait by the door, steer a ship, open a menu. Unlike a [Talking NPC](https://docs.tryllengine.com/v0.9/use-cases/talking-npc.html), the point here is not conversation. It is turning free-form speech into a command your game can run.

**Building blocks:** speech-to-text, a small language model, and either tool calls or a grammar. Text-to-speech is optional.

```
flowchart LR
    Mic["'Bravo, get behind<br/>the red truck'"] -->|STT| Text["Transcript"]
    Text --> SLM["SLM: understand"]
    SLM --> Cmd["move(unit=bravo,<br/>target=red_truck)"]
    Cmd --> Game["Your game checks<br/>and runs it"]
    Game --> Ack["'Moving!'<br/>(prewritten line)"]
```

The model's only job is understanding. Players do not say commands the same way twice ("Bravo, cover behind that truck", "get Bravo to the red truck", "Bravo, truck, now"), and a small model maps all of them to the same command. Your game does everything else: it checks the command makes sense, runs it, and plays the reply.

The characters' replies do not need a language model at all. A soldier answering "Moving!" can use your game's usual recorded lines, or a [`CannedResponse`](https://docs.tryllengine.com/v0.9/reference/nodes/canned-response.html) node picking from a list. Add [text-to-speech](https://docs.tryllengine.com/v0.9/how-to/add-voice-to-an-agent.html) only if you want generated replies.

______________________________________________________________________

## Two ways to get a command

|                           | Tool calls                                                                                                 | Grammar                                                                                                          |
| ------------------------- | ---------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **What the model writes** | A call to one of your tools, with typed arguments, such as `move(unit, target)`                            | One line that must match a pattern, such as `<action> <object>`                                                  |
| **Best for**              | Commands with different shapes and arguments                                                               | One flat vocabulary of actions and objects                                                                       |
| **How it fails**          | The model can call no tool when it should; that path is routed, not an error                               | The output always fits the pattern, but can be the wrong command                                                 |
| **Node**                  | [`ToolCall`](https://docs.tryllengine.com/v0.9/reference/nodes/tool-call.html)                             | [`Generate`](https://docs.tryllengine.com/v0.9/reference/nodes/generate.html) with a `grammar`                   |
| **Read**                  | [Define and handle tool calls](https://docs.tryllengine.com/v0.9/how-to/define-and-handle-tool-calls.html) | [Constrain output with a grammar](https://docs.tryllengine.com/v0.9/how-to/constrain-output-with-a-grammar.html) |

**Tool calls** fit most games. Each command is a tool with a clear description and only the arguments it really needs. When the player says "get Bravo behind the red truck", your game receives the tool name and its arguments as JSON.

**A grammar** fits a fixed vocabulary, such as `open door`, `take key`, `go north`. The grammar only allows lines in that form, so the output is always machine-readable. The grammar can be changed at runtime, so your game can rebuild it from what is on screen right now: only the units that exist, only the doors in this room.

Either way, the model guarantees the *form* of the command, never that it is the right one. Your game must still check it: is there a red truck, is Bravo alive, can Bravo reach it? When the answer is no, have the character say so ("Can't see a red truck, sir"), the same way a human squadmate would.

______________________________________________________________________

## Make it hear your words

Voice control fails first in speech recognition, not in the model. Game words are hard for it: unit call signs, invented place names, spell names.

- **Hotwords.** Give the speech recognizer a list of your game's words, and it will prefer them: "Aeltharion" instead of "Al Therion", "frostbolt" instead of "frost bolt". See [Bias voice input with hotwords](https://docs.tryllengine.com/v0.9/how-to/use-voice-input-hotwords.html). Not every speech model supports this: Parakeet and Zipformer do; Whisper, SenseVoice and Paraformer do not.
- **Push-to-talk.** For commands, a key to hold while speaking is more reliable than always listening. The player decides what is a command and what is chatter with friends.
- **Short words.** One-word commands such as "Fire!" are easy to clip. Keep a little silence padding around each recorded phrase. See [Use voice input](https://docs.tryllengine.com/v0.9/how-to/use-voice-input.html).

______________________________________________________________________

## Keep each command independent

A command agent usually should not remember earlier commands. Make it **stateless**: every phrase is understood on its own, and an old order can never leak into a new one. If a command depends on what the player sees, such as "attack *that* one", pass the current situation with each phrase, for example the selected target, as an [agent variable](https://docs.tryllengine.com/v0.9/concepts/agent-variables.html) rendered into the prompt.

______________________________________________________________________

## Choosing a model

Command parsing is one of the easier jobs for a small model. In our [tool-call comparison](https://docs.tryllengine.com/v0.9/how-to/compare-tool-call-models.html), on a suite that maps player speech to heist-game commands:

| Model                  | Commands passed |
| ---------------------- | --------------- |
| NVIDIA Orchestrator 8B | 20 / 20         |
| Ministral 3 14B        | 20 / 20         |
| Bonsai 8B (1-bit)      | 20 / 20         |
| Qwen 3.5 4B            | 18 / 20         |
| Llama 3.1 8B           | 17 / 20         |
| Llama 3.2 1B           | 14 / 20         |

Bonsai 8B in its 1-bit form scores 20 of 20 while needing about 1.9 GB of video memory, which leaves the rest of the GPU to your game.

**In the demo:** *CommandParser* (a stateless agent that turns any phrase into one of 16 `<action> <object>` commands with a grammar, on Llama 3.2 3B).

## Related

- [Use Cases](https://docs.tryllengine.com/v0.9/use-cases/index.html)
- [Tool calling](https://docs.tryllengine.com/v0.9/concepts/tool-calling.html)
- [Design a tool-call prompt](https://docs.tryllengine.com/v0.9/how-to/design-a-tool-call-prompt.html)
- [Constrained output (GBNF)](https://docs.tryllengine.com/v0.9/concepts/constrained-output.html)
- [STT and voice input](https://docs.tryllengine.com/v0.9/concepts/stt-and-voice-input.html)
