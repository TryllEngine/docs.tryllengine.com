# How-to Guides

Task recipes. Each page solves one concrete problem and shows a minimal working example in Unity, Unreal, C++, and Python where applicable. If you are new to Tryll, start with [First inference](https://docs.tryllengine.com/v0.9/getting-started/index.html); if you want the *why*, see [Concepts](https://docs.tryllengine.com/v0.9/concepts/index.html).

Guides marked ★ are the ones to read first in their area.

## Setup and connection

- [Run the Tryll Server](https://docs.tryllengine.com/v0.9/how-to/run-the-tryll-server.html) — install or build from source, edit `server-config.json`, start, watch logs.
- [Auto-launch the Server](https://docs.tryllengine.com/v0.9/how-to/auto-launch-server.html) — let the client spawn and manage the server process for you (Unity, Unreal, C++, Python).
- [Connect and Manage a Session](https://docs.tryllengine.com/v0.9/how-to/connect-and-manage-session.html) — connect, create the session, reconnect, clean shutdown.

## Editor tools

- [Test an Agent in the Editor](https://docs.tryllengine.com/v0.9/how-to/test-agent-in-editor.html) ★ — iterate on a workflow in the Unity/Unreal Chat window without entering Play/PIE.
- [Edit Workflows in the Unity Graph Editor](https://docs.tryllengine.com/v0.9/how-to/edit-workflows-in-graph-editor.html) and [in the Unreal Graph Editor](https://docs.tryllengine.com/v0.9/how-to/edit-workflows-in-unreal-graph-editor.html) — author a workflow visually: add nodes, wire exits, validate, save.
- [Manage Models in the Editor](https://docs.tryllengine.com/v0.9/how-to/manage-models-in-editor.html) — browse, download, load/unload, and register models for builds from the GUI.
- [Choose Where Models Run](https://docs.tryllengine.com/v0.9/how-to/choose-where-models-run.html) — put each model in RAM or VRAM, and understand why placement is a hint rather than a guarantee.
- [Use the Agent Log](https://docs.tryllengine.com/v0.9/how-to/use-agent-log.html) — the per-agent send/receive narrative in the editor: what your game asked for, and what came back.
- [Inspect Server Memory](https://docs.tryllengine.com/v0.9/how-to/inspect-server-memory.html) — Unity and Unreal Memory windows: live RAM/VRAM for the whole server process (Refresh only). The Knowledge window (Unity **Window ▸ Tryll ▸ Knowledge**, Unreal **Window ▸ Tryll ▸ Tryll Knowledge**) also edits experimental mutable stores: stores on the left, Filter or Query records in the center, and write, delete, clear, and per-store export/import on the right.
- [Compare Dialog Variants in the Lab](https://docs.tryllengine.com/v0.9/how-to/compare-dialog-variants.html) — run one scripted dialog across parameter variants and seeds, and read the spread instead of guessing from a single reply.
- [Test Tool Calls in Chat and the Dialog Lab](https://docs.tryllengine.com/v0.9/how-to/test-tool-calls-in-editor.html) — register a handler on the scene agent, talk to it in Chat, then script canned or live-tool Lab runs.
- [Agentic Coding With Tryll](https://docs.tryllengine.com/v0.9/how-to/agentic-coding-with-tryll.html) — have an AI coding agent build Tryll agents: `AGENTS.md`, the Unreal MCP tools, and how to write a prompt it can finish.

## Build an agent

- [Build a Chat Agent with a Graph](https://docs.tryllengine.com/v0.9/how-to/build-chat-agent-with-graph.html) ★ — compose a `Generate` + guardrail graph. Start here when writing your first agent.
- [Design an NPC Prompt](https://docs.tryllengine.com/v0.9/how-to/design-an-npc-prompt.html) ★ — measurement-backed recipes for writing a role-play `system_prompt`: what actually improves character dialogue, what provably does not, and how model choice changes the answer.
- [Design an Intent-Classifier Prompt](https://docs.tryllengine.com/v0.9/how-to/design-an-intent-classifier-prompt.html) — the `ClassifyIntentLLM` template contract (`{{intents_block}}` required; `{{letters}}` optional), how to write and order the labels, and how to tune `threshold`/`margin`.
- [Use Mustache Templates](https://docs.tryllengine.com/v0.9/how-to/use-mustache-templates.html) — control exactly how instructions, slots, variables, and retrieved chunks land in the prompt.
- [Stream Answers to a UI](https://docs.tryllengine.com/v0.9/how-to/stream-answers-to-ui.html) — wire streaming tokens into your application's view layer.
- [Send Multiple Answers in One Turn](https://docs.tryllengine.com/v0.9/how-to/send-multiple-answers-in-a-turn.html) — two speakers, or a thinking channel plus a reply, routed by `AnswerText.node_name`.

## Dialogue and turn control

- [Change Agent Parameters at Runtime](https://docs.tryllengine.com/v0.9/how-to/change-agent-parameters.html) — mutate a node's params (prompt, sampling, threshold, …) between turns, and which params are structural and cannot change.
- [Seed and Edit Dialog History](https://docs.tryllengine.com/v0.9/how-to/seed-and-edit-dialog-history.html) — append scripted user/assistant turns, write an opener, or retry by removing the last turn — all without running the graph.
- [Export and Import Dialog](https://docs.tryllengine.com/v0.9/how-to/export-and-import-dialog.html) — capture a full-fidelity snapshot and restore it from a game-owned save file.
- [Save and Load Agent State](https://docs.tryllengine.com/v0.9/how-to/save-and-load-agent-state.html) — coordinate quiescent multi-agent snapshots, variables, game state, atomic slot writes, strict restore, and KV-cache warm-up.
- [Cancel a Turn](https://docs.tryllengine.com/v0.9/how-to/cancel-a-turn.html) — stop an in-flight turn (a chat "Stop" button, or a rollback), keeping or discarding the partial reply.
- [Pause and Resume a Turn](https://docs.tryllengine.com/v0.9/how-to/pause-and-resume-turns.html) — suspend a turn between nodes and resume it after acting client-side.
- [Manage an Agent's KV Cache](https://docs.tryllengine.com/v0.9/how-to/manage-agent-kv-cache.html) — prefill, evict, and inspect language-model KV caches to trade memory against first-turn latency.
- [Run Many Agents on a Small GPU](https://docs.tryllengine.com/v0.9/how-to/run-many-agents-on-a-small-gpu.html) — size every LLM node's context window, then evict the agents nobody is talking to: the two levers that fit several NPC agents into limited VRAM.
- [Keep Long Conversations Coherent](https://docs.tryllengine.com/v0.9/how-to/keep-long-conversations-coherent.html) — add a `Summarize` node so ageing turns fold into a rolling summary instead of being trimmed out of the prompt.

## Knowledge and retrieval

- [Create a Simple RAG Assistant](https://docs.tryllengine.com/v0.9/how-to/create-simple-rag-assistant.html) ★ — the hero recipe: prepare a knowledge base, add a `Retrieve` node in front of `Generate`, see grounded answers.
- [Query Rewriting and HyDE for RAG](https://docs.tryllengine.com/v0.9/how-to/query-rewriting-for-rag.html) — rewrite the user's message before retrieval, while still answering what they actually asked. Covers both registers: a keyword **query** when your chunks read like questions, and a hypothetical answer (**HyDE**) when they read like statements.
- [Build an Intent-Driven NPC](https://docs.tryllengine.com/v0.9/how-to/build-intent-driven-npc.html) — classify the player's intent against a labelled knowledge base and route to tone-specific replies.
- [Classify Intent Without a Knowledge Base](https://docs.tryllengine.com/v0.9/how-to/classify-intent-without-a-knowledge-base.html) — the same NPC using `ClassifyIntentLLM`: describe each intent in a sentence, no embedding model and no corpus to author.

## Game state and control flow

The mental model for this section is the per-turn [slot blackboard](https://docs.tryllengine.com/v0.9/concepts/slots-and-inter-node-value-passing.html) and the per-agent [variable store](https://docs.tryllengine.com/v0.9/concepts/agent-variables.html).

- [Drive Prompts and Retrieval from Game State](https://docs.tryllengine.com/v0.9/how-to/drive-prompts-and-retrieval-from-game-state.html) ★ — inject live game state (level, mood, quests) into a prompt template and a `Retrieve` filter via agent variables, updated each turn with no `ChangeParam` round-trip.
- [Branch on a Slot](https://docs.tryllengine.com/v0.9/how-to/branch-on-a-slot.html) — conditional routing with the `Branch` node, including reacting to a verdict the model produced earlier in the same turn.
- [Add a Hidden Reasoning or Draft Step](https://docs.tryllengine.com/v0.9/how-to/add-a-hidden-reasoning-step.html) — an internal draft or "thinking" pass the conversation never remembers, using `send` and `history_role`.
- [Substitute Agent Variables in LLM Output](https://docs.tryllengine.com/v0.9/how-to/substitute-agent-variables-in-output.html) — have the model emit `__PRICE__`-style markers and replace them with live variable values before the answer, history, or TTS sees the text.
- [Filter LLM Output Artifacts](https://docs.tryllengine.com/v0.9/how-to/filter-llm-output-artifacts.html) — strip speaker prefixes and roleplay markup spans out of streamed output.
- [Keep Merchant Prices Deterministic](https://docs.tryllengine.com/v0.9/how-to/keep-merchant-prices-deterministic.html) ★ — worked example: game code owns the number, the model owns the wording, and the two never disagree.

## Structured output and tools

- [Define and Handle Tool Calls](https://docs.tryllengine.com/v0.9/how-to/define-and-handle-tool-calls.html) ★ — declare tool definitions on a `ToolCall` node, route on `tool_called` / `no_tool_called`, handle the call client-side, and hand the real result back to the model.
- [Test Tool Calls in Chat and the Dialog Lab](https://docs.tryllengine.com/v0.9/how-to/test-tool-calls-in-editor.html) — exercise that workflow in the editors without Play/PIE.
- [Compare Tool-Call Models](https://docs.tryllengine.com/v0.9/how-to/compare-tool-call-models.html) ★ — exact-accuracy on the Tryll-authored suites and the BFCL slice; pick a catalog model by the job (commands, log query, parallel calls), not by parameter count.
- [Design a Tool-Call Prompt](https://docs.tryllengine.com/v0.9/how-to/design-a-tool-call-prompt.html) ★ — measurement-backed schema and prompt recipes: name "unfiltered", put it first in the enum, and do not add parameters the model cannot fill.
- [Fix Tool-Call Text in Answers](https://docs.tryllengine.com/v0.9/how-to/fix-tool-call-text-in-answers.html) — the player sees `<|tool_call>…` instead of dialogue: find which node wrote it, and fix the prompt, the routing, or the tool node.
- [Constrain Output with a Grammar](https://docs.tryllengine.com/v0.9/how-to/constrain-output-with-a-grammar.html) — force a `Generate` node to emit exactly a fixed shape (command, choice, JSON) with a GBNF grammar; flip between command turn and free chat at runtime.

## Guardrails and safety

- [Use Canned Responses and Guardrails](https://docs.tryllengine.com/v0.9/how-to/use-canned-responses-and-guardrails.html) — short-circuit jailbreaks and off-topic prompts to scripted replies, before spending a model call.
- [Build an Immersion Guard](https://docs.tryllengine.com/v0.9/how-to/build-an-immersion-guard.html) — gate out-of-character and out-of-world player lines before they reach the NPC, and drop the rejected turn from history.

## Voice

- [Add Voice Output to an Agent](https://docs.tryllengine.com/v0.9/how-to/add-voice-to-an-agent.html) ★ — make an NPC speak with `GenerateAndSpeak` / `Speak` and a playback component.
- [Add Lip Sync to a Character](https://docs.tryllengine.com/v0.9/how-to/add-lip-sync-to-a-character.html) — move a character's mouth with its voice using the server's viseme track.
- [Clone a Voice from an Audio Sample](https://docs.tryllengine.com/v0.9/how-to/clone-a-voice.html) — point a voice profile at a short reference WAV to speak in that voice.
- [Design a Voice Clip with Qwen3-TTS](https://docs.tryllengine.com/v0.9/how-to/design-a-voice-with-qwen3-tts.html) — make a reference clip from a text description, without an actor.
- [Add Emotions to a Voice](https://docs.tryllengine.com/v0.9/how-to/add-emotions-to-a-voice.html) — one clip per mood as delivery modes, switched at runtime.
- [Use Voice Input](https://docs.tryllengine.com/v0.9/how-to/use-voice-input.html) — create a VoiceInput handle, stream PCM audio, receive transcripts, push-to-talk vs. hands-free.
- [Bias Voice Input with Hotwords](https://docs.tryllengine.com/v0.9/how-to/use-voice-input-hotwords.html) — nudge the STT decoder toward game-specific proper nouns, spell names, and other lexicon items without retraining the model.
- [Add Lip Sync to a MetaHuman](https://docs.tryllengine.com/v0.9/how-to/add-lip-sync-to-metahuman.html) (Unreal) — make a MetaHuman's mouth follow an agent's TTS voice with the separate Tryll MetaHuman Lip Sync plugin, using Epic's neural model or the viseme track.

## Models and shipping

- [Use Your Own Local Model](https://docs.tryllengine.com/v0.9/how-to/use-your-own-local-model.html) — register a GGUF file that lives on disk without a Hugging Face download.
- [Use a LoRA Adapter](https://docs.tryllengine.com/v0.9/how-to/use-a-lora-adapter.html) — run an LLM node with a fine-tuned GGUF adapter, and tune how strongly it applies.
- [Pin and Unpin Models](https://docs.tryllengine.com/v0.9/how-to/pin-and-unpin-models.html) — keep a model warm (`LoadModelRequest`) or release it (`UnloadModelRequest`).
- [Verify Required Models Before Play](https://docs.tryllengine.com/v0.9/how-to/verify-models-before-play.html) — gate Play/PIE on every registered model being downloaded first, and fail the build when a workflow references a model that will not ship.
- [Estimate Memory Footprint](https://docs.tryllengine.com/v0.9/how-to/estimate-memory-footprint.html) — work out the RAM and VRAM a model line-up needs before you ship, so it fits your target hardware. Check the live numbers in [Inspect Server Memory](https://docs.tryllengine.com/v0.9/how-to/inspect-server-memory.html).
- [Ship Storage Folders for Builds](https://docs.tryllengine.com/v0.9/how-to/ship-storage-folders-for-builds.html) — lay out canned-response / guardrail / RAG / hotword files so they travel with a packaged build and resolve against the process storage root.
- [Ship an Agent in Another Language](https://docs.tryllengine.com/v0.9/how-to/ship-in-another-language.html) — set one session language and have speech, speech recognition, content files and prompt directives all follow it; plus what the language setting deliberately does *not* do.
- [Localize Prompts with Your Localization System](https://docs.tryllengine.com/v0.9/how-to/localize-prompts.html) — tick Localized on a prompt, store a Text ID, and let Unity or Unreal ask your string table for the text before the agent is created.

Downloading and registering models from the editor GUI is covered in [Manage Models in the Editor](https://docs.tryllengine.com/v0.9/how-to/manage-models-in-editor.html).

## Related

- [Concepts](https://docs.tryllengine.com/v0.9/concepts/index.html) — the mental model behind each recipe.
- [Reference](https://docs.tryllengine.com/v0.9/reference/index.html) — field-level detail for every knob.
