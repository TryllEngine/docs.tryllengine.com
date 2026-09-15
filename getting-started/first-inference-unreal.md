# First Inference in Unreal

Unpack a Tryll release, drop the plugin into your Unreal project (the server is bundled inside it), set a few project settings, download a model through the Model Manager, and watch a single `Generate` node print its answer to the screen during PIE — no chat UI required.

Working with an AI coding agent?

The Unreal plugin ships an **`AGENTS.md`** at its root. Attach it to your agent's context (Claude Code, Cursor, Copilot, Codex, …) before asking it to write Tryll code — it carries the graph/slot mental model and the ways Tryll differs from a cloud LLM SDK. Agents that fetch docs themselves can start from [`llms.txt`](https://docs.tryllengine.com/llms.txt).

## What you'll build

A blank actor with a one-node workflow. You download a model up front in the Model Manager, then the plugin auto-launches a local Tryll server, connects, and creates the session for you; the agent component creates itself once the session is ready. The actor runs one inference turn with the prompt *"In one sentence: what is Tryll?"* and prints the answer to the viewport and the Output Log.

## Before you start

- **Unreal Engine 5.7** and a blank **C++** project (Blueprint-only projects can't compile the plugin source).
- **~5 GB free disk space** for the downloaded model and a working internet connection on first run.
- Familiarity with creating a Blueprint actor and binding events.

The Unreal plugin bundles the server and spawns it for you, so installing the plugin is the whole setup.

______________________________________________________________________

## Step 1 — Get the distribution

1. Download `tryll-unreal-<version>.7z` from the releases page and extract it somewhere temporary (e.g. `C:\Downloads\tryll\`).

1. Inside is the `UnrealPlugin/` folder — **the server is bundled inside it**, so this one folder is everything you install:

   ```
   UnrealPlugin/                     <-- everything you need for Unreal
   ├── TryllClient.uplugin
   ├── Generated/messages_generated.h
   ├── Source/
   └── Binaries/ThirdParty/TryllServer/Default/  <-- bundled server
       ├── tryll_server.exe
       ├── llama.dll, ggml*.dll, ggml-vulkan.dll
       └── data/
           ├── server-config.json
           ├── models.json
           ├── default-canned-responses.txt
           └── default-guardrail-patterns.txt
   ```

The archive ships **no model weights** — you download the models your graph references up front through the Model Manager (Step 4). `CreateAgent` fails fast if a referenced model is not already on disk; the server does **not** download it for you at agent-creation time. Downloaded weights are cached under `data/.app-data/models/` beside the bundled exe.

How the archive is built

`ci/scripts/release.py` builds the production server, then stages the Unreal plugin (FlatBuffers codegen + the server bundled into its `Binaries/ThirdParty/TryllServer/Default/` folder) and zips it into a per-engine archive. The server's `data/server-config.json` is patched with production overrides and `models.json` is filtered to the shipping audience. The Unity package ships separately as `tryll-unity-<version>.7z`.

______________________________________________________________________

## Step 2 — Install the Unreal plugin

Copy the `UnrealPlugin/` folder into your project as `<YourProject>/Plugins/TryllClient/`:

```
<YourProject>/
├── Plugins/
│   └── TryllClient/
│       ├── TryllClient.uplugin
│       ├── Generated/messages_generated.h
│       ├── Source/TryllClient/{Public,Private}/
│       └── Binaries/ThirdParty/TryllServer/Default/  <-- server rides along
└── <YourProject>.uproject
```

The bundled server ships inside the plugin, so installing the plugin places the server alongside it. The plugin resolves it at `<Plugin>/Binaries/ThirdParty/TryllServer/<variant>/tryll_server.exe` via the plugin directory, so the path is identical in the editor and in packaged builds (a shipped plugin has just the `Default` variant). The auto-launcher runs it with **CWD set to the exe's own folder**, so `data/` sits next to it and downloaded GGUF weights are cached under `data/.app-data/models/`.

Then:

1. Right-click `<YourProject>.uproject` → **Generate Visual Studio project files**.
1. Open the solution and build in **Development Editor**.
1. Launch the editor. **Edit → Plugins → Project** lists **Tryll Client** as enabled.

______________________________________________________________________

## Step 3 — Configure Tryll Client in Project Settings

Open **Edit → Project Settings → Plugins → Tryll Client** and set:

| Setting                                                    | Value              | Why                                                                                                                                                   |
| ---------------------------------------------------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Auto Launch Server**                                     | `true` (default)   | Subsystem spawns the bundled server on `Initialize`. Uncheck to run a server yourself.                                                                |
| **Llama.cpp backend**                                      | `Vulkan` (default) | Portable GPU API. Set `Auto` to prefer CUDA on NVIDIA (also ships CUDA DLLs in a packaged build).                                                     |
| **Auto Connect**                                           | `true` (default)   | Subsystem opens the session on `Initialize` (connect-retry covers server startup). Uncheck to call `Connect` yourself.                                |
| **Auto Create Session**                                    | `true` (default)   | On connect, the subsystem creates the session from the models you registered — no Blueprint wiring needed. Uncheck to call `Create Session` yourself. |
| **Connect Max Attempts** / **Connect Retry Delay Seconds** | defaults           | Subsystem retries while the freshly-spawned server warms up.                                                                                          |
| **Game Name**                                              | e.g. `my-game`     | Short slug sent to the server on every session for telemetry grouping. Please fill it.                                                                |

Settings are persisted to `Config/DefaultGame.ini` under `[/Script/TryllClient.TryllRuntimeSettings]`.

The Model Manager is the primary way to get models

You download and register your model up front in the next step, so it is already on disk before the first turn. `CreateAgent` fails fast if the model is not already there — it does not download it for you — so shipped builds rely on the Model Manager's registration tiers. See [Manage Models in the Editor](https://docs.tryllengine.com/how-to/manage-models-in-editor.html).

______________________________________________________________________

## Step 4 — Download and register a model

Before your agent can run, it needs a model on disk. Download one with the Model Manager and register it to your project, then you can select it on the workflow in the next step.

1. Open **Window → Tryll → Tryll Model Manager**. The window launches the bundled server, connects, and lists every model the server knows about. Wait for the toolbar status to read **Ready · *N* models** (green).
1. In the list, pick a language model to start with:
   - **Recommended:** `Gemma 3 4B Instruct (Q4_K_M)` (~3 GB download).
   - **Smaller / quicker:** `Llama 3.2 3B Instruct (Q4_K_M)` (~2 GB download).
1. Select the row. In the **Model Properties** panel on the right, the **Status** section shows **State: Not downloaded**. Click **Download** — a progress bar tracks the bytes and the toolbar shows the active download count.
1. When the state turns to **Downloaded**, set **Build registration (this project)** to **Experimental** — that makes it available in the editor, which is all this tutorial needs. (**Production** additionally ships the files inside packaged builds; choose that when you're ready to ship.)
1. Close the Model Manager.

Registrations live in your project, written to `Config/DefaultGame.ini` via the **Tryll Model Manifest** developer settings. Every model-name dropdown in the editor now lists what you registered here.

Registration controls where a model is available

The tier you pick decides a model's reach: **Experimental** keeps it in the editor for development; **Production** ships it inside packaged builds beside the server. Node dropdowns list your Experimental and Production models, each with a tier badge. See [Manage Models in the Editor](https://docs.tryllengine.com/how-to/manage-models-in-editor.html).

______________________________________________________________________

## Step 5 — Create a workflow asset and add the demo actor

1. In the **Content Browser**, right-click → **Tryll → Workflow Asset**. Name it `DemoWorkflow`.
1. Open `DemoWorkflow`. In the Details panel configure the graph:
   - Under **Nodes**, add one entry and set **Name** = `answer`.
   - On that node's **Params**, pick **Generate** from the type picker — that makes it a Generate node and expands its parameter fields below. Leave them at their defaults; the **Default Exit** field stays empty, which routes the turn to END.
   - Set **Start Node** = `answer`.
   - Set **Default Model Name** to the model you registered in Step 4.
1. Create a blank Blueprint actor — call it `BP_TryllDemoActor`.
1. Add a **Tryll Agent Component** to it.
1. Assign `DemoWorkflow` to the **Workflow Asset** slot on the component.
1. **Leave Auto Create On Connect ticked** (the default). The component waits until the session is created before creating the agent, and queues itself if it spawns earlier — so it manages its own lifecycle.
1. Drop `BP_TryllDemoActor` into the level.

______________________________________________________________________

## Step 6 — Wire the Blueprint

With **Auto Launch Server**, **Auto Connect**, and **Auto Create Session** on (Step 3), the subsystem launches the server, opens the connection, and creates the session from your settings — all by itself. The agent component then creates itself as soon as the session is ready. So the only thing left to wire is *what to do when the agent answers*.

Open `BP_TryllDemoActor`'s Event Graph:

```
flowchart LR
    OAR[OnAgentReady] --> SM["Send Message<br>In one sentence: what is Tryll?"]
    OAT[OnAnswerText] --> P1["Print String: Text"]
    OTC[OnTurnComplete] --> P2["Print String: Done."]
```

Select the **Tryll Agent Component** in the Components panel, then add each event from the **Events** section at the bottom of the Details panel:

1. **On Agent Ready** → **Send Message** with `"In one sentence: what is Tryll?"`.
1. **On Answer Text (Node Name, Text, bIsFinal)** → **Print String** — wire `Text` into **In String**, set `Print to Screen = true`, `Print to Log = true`, `Duration = 10`. The `Generate` node streams by default, so this fires once per chunk (`Text` is a delta) and `bIsFinal` is `true` on the last chunk; `Node Name` names the node that produced the chunk (`answer` here).
1. **On Turn Complete (Status, DebugInfo, TokensGenerated)** → another **Print String** that prints `"Done."` (or **Enum to String** on `Status`).

That's the whole graph — no `Connect`, no `Create Session`, no `Create Agent`. Those are the subsystem's job now.

Want the whole reply as one line?

Streaming (`Send = Streamed`) is the `Generate` node's default. To get a single frame with the complete reply instead of per-chunk deltas, either bind **On Answer Full** (fires once after `OnTurnComplete` with the full accumulated text) or set the node's **Send** to **Whole** in Step 5.

Driving the session yourself

For full control — a loading screen, multiple sessions, or non-default engines per actor — untick **Auto Connect** / **Auto Create Session** and call `Connect` then `Create Session` (or `Create Session From Settings`) yourself. `Create Agent` still waits for the session to be ready, so the old connect-vs-create race can't happen either way.

______________________________________________________________________

## Step 7 — Test without entering PIE

Before pressing Play, use the **Chat Window** to verify the agent responds — no Blueprint wiring required:

1. Open **Window → Tryll → Tryll Chat**.
1. Select the actor that has your `UTryllAgentComponent` (with `DemoWorkflow` assigned on the component).
1. Click **Start** — the panel launches the server, connects, and creates an editor agent from that component. The status label turns green: **Ready**.
1. Type a message and press **Enter**. You should see a streamed reply.
1. Click **Stop** when done.

See [Test an Agent in the Editor](https://docs.tryllengine.com/how-to/test-agent-in-editor.html) for the full walkthrough including troubleshooting.

______________________________________________________________________

## Step 8 — Play

Press **Play**. On the **first** run you'll see:

1. A few seconds while the server starts and the plugin connects.

1. The model you downloaded in Step 4 loads from disk into memory — a few seconds the first time it's used in a session.

1. The answer streams to the viewport and the Output Log, one chunk per `OnAnswerText` event, followed by your `OnTurnComplete` marker:

   ```
   LogBlueprintUserMessages: [BP_TryllDemoActor] Tryll is a local
   LogBlueprintUserMessages: [BP_TryllDemoActor]  small-language-model
   LogBlueprintUserMessages: [BP_TryllDemoActor]  inference server.
   LogBlueprintUserMessages: [BP_TryllDemoActor] Done.
   ```

The `Generate` node streams by default (`Send = Streamed`), so each `OnAnswerText` chunk carries a delta; the last chunk has `bIsFinal = true`, then your `OnTurnComplete` handler prints `Done.`. For a single full-reply frame instead, bind **On Answer Full** or set the node's **Send** to **Whole** (see the tip in Step 6).

Subsequent runs skip the download and load steps — the model stays in the server process, and the `.app-data/models/` cache survives across PIE sessions.

______________________________________________________________________

## Where to see what's happening

Open **Window → Output Log** and use the **Categories** filter:

| Category                   | Source                                                                        | Useful for                                                                                                                                             |
| -------------------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `LogTryllServer`           | `tryll_server.exe` stdout (drained by the subsystem and re-emitted in Unreal) | `[Listener] Listening on 127.0.0.1:<port>` (auto-launch binds loopback on an OS-chosen port), `ConnectionReady`, model load lines, server-side errors. |
| `LogTryll`                 | Plugin (subsystem, agent component, connection thread)                        | Connect attempts, `CreateSession` outcome, agent lifecycle, `DownloadProgress`.                                                                        |
| `LogBlueprintUserMessages` | `Print String` calls                                                          | The answer text, your "Done." marker.                                                                                                                  |

The server's output funnels through the Unreal Output Log, so a single window shows everything.

A rotating server log file is also written next to the bundled server, at `<Plugin>/Binaries/ThirdParty/TryllServer/Default/data/.app-data/logs/tryll.log` (production log mode is the default in the shipped `server-config.json`).

______________________________________________________________________

## What you built

- A session connected to a Tryll server that the plugin spawned for you.
- One agent with a one-node graph.
- Model output surfaced to Blueprint as `OnAnswerText` events, which stream token-by-token by default (`Send = Streamed`); set the node's **Send** to **Whole**, or bind **On Answer Full**, for a single full-reply frame.

Everything the plugin exposes in Blueprint — connection, session, agents, models, string storages, tool calls — is listed in the [Blueprint Catalog](https://docs.tryllengine.com/reference/client-api/unreal/blueprint-catalog.html).

## Where to go next

- **[Test an Agent in the Editor](https://docs.tryllengine.com/how-to/test-agent-in-editor.html)** — iterate on your workflow without entering PIE.
- **[How-to Guides](https://docs.tryllengine.com/how-to/index.html)** — RAG, tool calls, guardrails, model management.
- **[Concepts](https://docs.tryllengine.com/concepts/index.html)** — the mental model behind the graph abstraction.
- **[Unreal C++ API](https://docs.tryllengine.com/reference/client-api/unreal/cpp/index.html)** and the [Blueprint Catalog](https://docs.tryllengine.com/reference/client-api/unreal/blueprint-catalog.html).
- **[Auto-launch the Server](https://docs.tryllengine.com/how-to/auto-launch-server.html)** — every Unreal-side server-launch knob explained.
- **[Manage models in the editor](https://docs.tryllengine.com/how-to/manage-models-in-editor.html)** — download, register, and pin models with explicit progress.

## Troubleshooting

- **`Tryll server exe not found at '...'` in `LogTryll/Error`** — no variant folder under `<Plugin>/Binaries/ThirdParty/TryllServer/` resolves. From a release archive the `Default` variant ships populated; from a source build, run `cmake --build` of the server so the committed `Default` symlink resolves to your build.
- **`OnConnectionChanged(false)` fires repeatedly** — the server crashed or didn't start. Check `LogTryllServer` for the failure reason and `data/.app-data/logs/tryll.log` (under the bundled server's folder) as a fallback.
- **`OnCreateSessionComplete(false)`** — usually a server-side error reading `models.json` or initialising the inference engine. Check `LogTryllServer`.
- **`Default Model Name` dropdown is empty** — register a model first. Open **Window → Tryll → Tryll Model Manager**, download one, and set its **Build registration** to **Experimental** or **Production**; it then appears in the dropdown.
- **`OnError` fires during `CreateAgent`** — read `ErrorMessage`. Common causes: `Default Model Name` is empty or names a model that isn't yet on disk (fails fast — download it via the Model Manager first), or a route targets a node name that doesn't exist.
- **`CreateAgent` takes several seconds even after the model is downloaded** — normal on the first agent using a given model in a session; the server is loading it from disk into RAM/VRAM. Subsequent agents on the same model are instant.
- **Nothing prints after `Send Message`** — bind both **On Answer Text** *and* **On Turn Complete**. With the default `Send = Streamed`, `OnAnswerText` fires once per chunk (each `Text` is a delta), the last chunk has `bIsFinal = true`, and `OnTurnComplete` follows. Set the node's **Send** to **Whole** to get a single full-reply frame.
