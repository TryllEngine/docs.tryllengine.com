# Report a Bug

A good report has the right evidence attached. Which evidence depends on the kind of bug: a crash needs a crash dump, a wrong answer needs an Agent Log export. This page helps you pick the kind, collect what it needs, and post it in the Discord **Bug Hunt** channel.

Scope

This page covers the **Unity and Unreal editor plugins**. Bugs in a packaged game build store their logs in a folder your game chooses, not in the places described here.

## 1. Note the basics

Every report needs these, whatever the bug:

- **Versions.** Tryll plugin and server version. In Unity, **Window → Tryll → Help & Community** shows both on one line; copy it. In Unreal, **Edit → Plugins** shows the plugin version, and the server log's first lines show the server version.
- **Engine.** Unity or Unreal, and its version.
- **Machine.** Windows version, GPU, and GPU driver version.
- **Model.** Which model or models the agent used (name as shown in the Model Manager).
- **Steps to reproduce.** What you did, what you expected, and what happened instead. Say whether it happens every time or only sometimes.

## 2. Identify the kind of bug

| What you saw                                                                                                 | Kind                       | Go to                                                     |
| ------------------------------------------------------------------------------------------------------------ | -------------------------- | --------------------------------------------------------- |
| The server stopped: Chat and other windows lost the connection, or the editor reports the server exited      | Server crash               | [Server crash](#server-crash)                             |
| A request failed with an error message or code: agent not created, model failed to load                      | Server error               | [Server error](#server-error)                             |
| The agent answered, but wrongly: wrong route or intent, missing tool call, broken text, ignored instructions | Unexpected agent behaviour | [Unexpected agent behaviour](#unexpected-agent-behaviour) |
| A download stalls or fails, or the server runs out of memory                                                 | Models and memory          | [Models and memory](#models-and-memory)                   |
| Speech is not recognised, voice output is missing or garbled, lip sync is off                                | Voice                      | [Voice](#voice)                                           |
| A Tryll editor window breaks, or the editor console shows an exception from Tryll                            | Editor plugin              | [Editor plugin](#editor-plugin)                           |
| Answers are much slower than usual, or the game stutters while an agent runs                                 | Performance                | [Performance](#performance)                               |

If more than one row fits, collect the evidence for each.

## 3. Collect the evidence

### Where the server logs are

The server writes one log file per run, `tryll_<date-time>.log`, into a logs folder for your project. Crash dumps go into **the same folder**.

To open it, use **Window → Tryll → Open Logs Folder** (Unity) or **Window → Tryll → Open Tryll Logs Folder** (Unreal). The **Help & Community** window has the same button. This works even when the server is not running. If the project has no logs yet, the parent `projects` folder opens instead.

To find it by hand, open:

```
%LOCALAPPDATA%\Tryll\projects\
```

and pick the folder whose name starts with your project name, then its `logs` subfolder. The part after the name is a hash of the project path, so a project you moved or copied has a second folder.

Attach the log file from **the run where the bug happened**, which is usually the newest one. Check the time in its name against when you saw the bug.

### Where the editor logs are

In the **Console** window, open the **⋮** menu and choose **Open Editor Log**. The file is `%LOCALAPPDATA%\Unity\Editor\Editor.log`. It is overwritten each time Unity starts, so copy it before you restart the editor.

`<YourProject>\Saved\Logs\<YourProject>.log`. Older runs are kept beside it with a date-time suffix.

### Server crash

1. Open the logs folder.
1. Find the crash dump: `tryll-crash-<version>-<commit>-<date-time>.dmp`. Take the newest one, and check that its time matches the crash.
1. Find the server log from the same run. Its last lines usually say where the dump was written.
1. Attach **both** files, plus the editor log.

Crash dumps are never deleted automatically. Delete old ones once you have reported them.

A crash dump contains memory

A dump holds part of the server's memory at the moment of the crash. That can include prompts, lore, and player input from the running agents. If you do not want that in a public channel, see [Private information](#private-information).

### Server error

1. Copy the error message exactly as shown, including the error code if there is one. The codes are listed in [Error Codes](https://docs.tryllengine.com/v0.10/reference/error-codes.html).
1. Attach the server log from that run and the editor log.
1. If the error came from creating an agent, attach the workflow asset too.

### Unexpected agent behaviour

The [Agent Log](https://docs.tryllengine.com/v0.10/how-to/use-agent-log.html) records what the editor sent and what each agent sent back, with turn diagnostics: route, prompt, retrieval, and tool calls. Export it and attach it.

1. Open the Agent Log **before** the agents are created: **Window → Tryll → Agent Log** (Unity) or **Window → Tryll → Tryll Agent Log** (Unreal). Agents created earlier have no diagnostics, so restart Play or reopen Chat after opening the window.
1. Reproduce the bug.
1. Click **Export** and save the JSON file.
1. Attach the export, the workflow asset the agent uses, and, in your description, the turn that went wrong: what you sent, what you expected, and what came back.

An export contains the conversation

The file includes player input, prompts, lore, and tool arguments. If you do not want that in a public channel, see [Private information](#private-information).

### Models and memory

1. Name the model and where it came from: the built-in catalog or [your own model](https://docs.tryllengine.com/v0.10/how-to/use-your-own-local-model.html).
1. Attach the server log from that run.
1. For memory problems, open the [Memory window](https://docs.tryllengine.com/v0.10/how-to/inspect-server-memory.html), click **Refresh**, and attach a screenshot. Mention how many agents were running.

### Voice

1. Attach an [Agent Log](#unexpected-agent-behaviour) export of the turns that went wrong. It shows the final speech transcripts and which node produced each piece of audio.
1. Attach the server log from that run.
1. Name the voice model, and the speech-recognition model for input problems.
1. If you can, attach a short recording of what was said or what was played.

### Editor plugin

1. Attach a screenshot of the broken window.
1. Attach the editor log, or copy the full exception with its stack trace from the console.

### Performance

1. Attach an [Agent Log](#unexpected-agent-behaviour) export. Each turn's diagnostics include its timing.
1. Name the model and its placement (GPU or CPU; see [Choose Where Models Run](https://docs.tryllengine.com/v0.10/how-to/choose-where-models-run.html)), and how many agents ran at once.
1. Attach the server log from that run.

## 4. Send the report

Join the [Tryll Discord](https://discord.com/invite/DTKycjPpzs). That is the same link as **Join our Discord** on the Welcome window: **Window → Tryll → Welcome** in Unity, **Window → Tryll → Tryll Welcome** in Unreal.

Post the report in **Bug Hunt**. The channel is public: anyone in the Discord can read the message and every file attached to it.

Post one message per bug. Use this template, and attach the files from step 3 to the same message:

```
Title: <one line: what is wrong>

Kind: <server crash | server error | agent behaviour | models and memory | voice | editor plugin | performance>
Versions: <line copied from Help & Community>
Engine: <Unity or Unreal, version>
Machine: <Windows version, GPU, driver version>
Model: <model name(s)>

Steps to reproduce:
1.
2.

Expected:
Actual:
Happens: <every time | sometimes | once>

Attached: <list of files>
Held back: <none, or which files you left out because they contain private information>
```

### Private information

Send the logs, the crash dump, and the other files from step 3. A crash dump, a server log, and an Agent Log export can contain prompts, lore, and player input.

If you do not want a file in a public channel because it contains private information, leave that file out. Say so on the **Held back** line. We will contact you to get that information from you.

## Related

- [Use the Agent Log](https://docs.tryllengine.com/v0.10/how-to/use-agent-log.html) — capture and export agent traffic
- [Error Codes](https://docs.tryllengine.com/v0.10/reference/error-codes.html) — what each server error code means
- [Inspect Server Memory](https://docs.tryllengine.com/v0.10/how-to/inspect-server-memory.html) — RAM and VRAM for the whole server
- [Run the Tryll Server](https://docs.tryllengine.com/v0.10/how-to/run-the-tryll-server.html) — server configuration and logs
