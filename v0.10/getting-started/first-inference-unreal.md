# First Inference in Unreal

Download the Tryll plugin zip for your engine version, unzip it into your project (the server is bundled inside it), set a few project settings, download a model through the Model Manager, and watch a single `Generate` node print its answer to the screen during PIE — no chat UI required.

Working with an AI coding agent?

The Unreal plugin ships an **`AGENTS.md`** at its root. Attach it to your agent's context (Claude Code, Cursor, Copilot, Codex, …) before asking it to write Tryll code — it carries the graph/slot mental model and the ways Tryll differs from a cloud LLM SDK. Agents that fetch docs themselves can start from [`llms.txt`](https://docs.tryllengine.com/v0.10/llms.txt).

## What you'll build

A blank actor with a one-node workflow. You download a model up front in the Model Manager, then the plugin auto-launches a local Tryll server, connects, and creates the session for you; the agent component creates itself once the session is ready. The actor runs one inference turn with the prompt *"In one sentence: what is Tryll?"* and prints the answer to the viewport and the Output Log.

## Before you start

- **Unreal Engine 5.7 or 5.8** on Windows x64, and a blank project. A Blueprint-only project is fine: the release zips carry prebuilt plugin binaries. (Packaging a game later needs Visual Studio 2022 with the "Game development with C++" workload, because packaging compiles the plugins for your game.)
- **~5 GB free disk space** for the downloaded model and a working internet connection on first run.
- Familiarity with creating a Blueprint actor and binding events.

The Unreal plugin bundles the server and spawns it for you, so installing the plugin is the whole setup.

______________________________________________________________________

## Step 1 — Download the plugin

1. Open the [Tryll Unreal releases](https://github.com/TryllEngine/tryll-unreal-plugin/releases) and pick the newest version.
1. Download `TryllClient-<version>-UE<engine>-Win64.zip` for your engine, for example `TryllClient-1.2.3-UE5.7-Win64.zip`. The other zips on the page are optional plugins (`TryllMetaHumanLipSync`, `TryllMCP`) and the standalone server, which you do not need here: **the server is bundled inside the `TryllClient` zip**.

The zip ships **no model weights** — you download the models your graph references up front through the Model Manager (Step 4). `CreateAgent` fails fast if a referenced model is not already on disk; the server does **not** download it for you at agent-creation time. In Client Dev, downloaded weights land under `<Project>/TryllStorage/models/`.

______________________________________________________________________

## Step 2 — Install the Unreal plugin

Unzip the zip into your project's `Plugins/` folder, so you get `<YourProject>/Plugins/TryllClient/`:

```
<YourProject>/
├── TryllStorage/                    <-- default storage root (outside Content/)
├── Plugins/
│   └── TryllClient/
│       ├── TryllClient.uplugin
│       ├── Source/                  <-- plugin source (built for you already)
│       ├── Binaries/Win64/          <-- prebuilt editor binaries for this engine
│       └── Binaries/ThirdParty/TryllServer/Default/  <-- server rides along
└── <YourProject>.uproject
```

The bundled server ships inside the plugin, so installing the plugin places the server alongside it. The plugin resolves it at `<Plugin>/Binaries/ThirdParty/TryllServer/<variant>/tryll_server.exe` via the plugin directory, so the path is identical in the editor and in packaged builds (a shipped plugin has just the `Default` variant). Auto-launch passes `--mode client-dev`, `--storage-root` (default `<Project>/TryllStorage/`), and related project flags. Models download into `TryllStorage/models/`.

Then open the project. It opens without compiling: the zip carries prebuilt binaries for this engine version. **Edit → Plugins → Project** lists **Tryll Client** as enabled.

Building from source, or a partner `.7z` package

The [plugin repository](https://github.com/TryllEngine/tryll-unreal-plugin) holds the plugin source without binaries or server. Copy `TryllClient/` into a **C++** project's `Plugins/` folder, unzip `tryll-server-<version>-win64-lite.zip` (or `-full` for CUDA) from the same release into `Plugins/TryllClient/Binaries/ThirdParty/TryllServer/Default/`, then build the project in **Development Editor**. A partner `.7z` package's `UnrealPlugin/` folder goes to `Plugins/TryllClient/` the same way.

### Accept the licence

When the editor starts, the **Tryll Welcome** tab opens. You can also open it from **Window → Tryll → Tryll Welcome**. It has two setup steps:

1. **Licence.** Click **Accept licence**. The editor starts the server only after you accept the Tryll Runtime Licence. The licence includes telemetry, which is always on in the editor. The game you ship sends no telemetry. **Telemetry** opens the [Telemetry](https://docs.tryllengine.com/v0.10/reference/telemetry.html) page. **Read the licence** shows the full text from the plugin's `LICENSE.md`.
1. **Server.** The plugin bundles its server, so this step is done as soon as you accept.

Tryll windows that were waiting for the server connect as soon as you accept. **Withdraw** takes the acceptance back and stops the editor's server.

______________________________________________________________________

## Step 3 — Configure Tryll Client in Project Settings

Open **Edit → Project Settings → Plugins → Tryll Client** and set:

| Setting                                                    | Value                    | Why                                                                                                                                                                                                           |
| ---------------------------------------------------------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Auto Launch Server**                                     | `true` (default)         | Subsystem spawns the bundled server on `Initialize`. Uncheck to run a server yourself.                                                                                                                        |
| **Llama.cpp backend**                                      | `Vulkan` (default)       | Portable GPU API. Set `Auto` to prefer CUDA on NVIDIA (also ships CUDA DLLs in a packaged build).                                                                                                             |
| **Auto Connect**                                           | `true` (default)         | Subsystem opens the session on `Initialize` (connect-retry covers server startup). Uncheck to call `Connect` yourself.                                                                                        |
| **Auto Create Session**                                    | `true` (default)         | On connect, the subsystem creates the session from the models you registered — no Blueprint wiring needed. Uncheck to call `Create Session` yourself.                                                         |
| **Connect Max Attempts** / **Connect Retry Delay Seconds** | defaults                 | Subsystem retries while the freshly-spawned server warms up.                                                                                                                                                  |
| **Game Name**                                              | e.g. `my-game`           | Human project label. Sent as the session name and as `--project-name`. Telemetry groups by a hidden project id; see [Telemetry](https://docs.tryllengine.com/v0.10/reference/telemetry.html). Please fill it. |
| **Telemetry Level**                                        | `Basic` (default)        | `Turns` or `Full` send more while debugging. Takes effect the next time the editor launches the server.                                                                                                       |
| **Storage Data Folder**                                    | `TryllStorage` (default) | Project folder **outside** `Content/`. The plugin passes `--storage-root`.                                                                                                                                    |

Settings are persisted to `Config/DefaultGame.ini` under `[/Script/TryllClient.TryllRuntimeSettings]`.

**Project Settings → Plugins → Tryll Client** with the defaults: auto-launch and auto-connect on, Vulkan backend, port 9100, and `TryllStorage` as the storage folder.

The editor sends content-free telemetry at the `basic` level. To send turn timings or conversation text while debugging, see [Telemetry](https://docs.tryllengine.com/v0.10/reference/telemetry.html). Packaged players send nothing.

The Model Manager is the primary way to get models

You download and register your model up front in the next step, so it is already on disk before the first turn. `CreateAgent` fails fast if the model is not already there — it does not download it for you — so shipped builds rely on the Model Manager's registration tiers. See [Manage Models in the Editor](https://docs.tryllengine.com/v0.10/how-to/manage-models-in-editor.html).

______________________________________________________________________

## Step 4 — Download and register a model

Before your agent can run, it needs a model on disk. Download one with the Model Manager and register it to your project, then you can select it on the workflow in the next step.

1. Open **Window → Tryll → Tryll Model Manager**. The window launches the bundled server, connects, and lists every model the server knows about. Wait for the toolbar status to read **Ready · *N* models** (green).
1. In the list, pick a language model to start with:
   - **Recommended:** `Gemma 4 E4B Instruct (Q4_K_M)` (~5.4 GB download, about 6 GB of VRAM). The demo workflows use it.
   - **Smaller / quicker:** `Llama 3.2 3B Instruct (Q4_K_M)` (~2 GB download, about 2.6 GB of VRAM). Pick this one on a 6 GB card.
1. Select the row. In the **Model Properties** panel on the right, the **Status** section shows **State: Not downloaded**. Click **Download** — a progress bar tracks the bytes and the toolbar shows the active download count.
1. When the state turns to **Downloaded**, set **Build registration (this project)** to **Experimental** — that makes it available in the editor, which is all this tutorial needs. (**Production** additionally ships the files inside packaged builds; choose that when you're ready to ship.)
1. Close the Model Manager.

Registrations live in your project, written to `Config/DefaultGame.ini` via the **Tryll Model Manifest** developer settings. Every model-name dropdown in the editor now lists what you registered here.

Registration controls where a model is available

The tier you pick decides a model's reach: **Experimental** keeps it in the editor for development; **Production** ships it inside packaged builds beside the server. Node dropdowns list your Experimental and Production models, each with a tier badge. See [Manage Models in the Editor](https://docs.tryllengine.com/v0.10/how-to/manage-models-in-editor.html).

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
1. **On Answer Text (Node Name, Text, bIsFinal)** → **Print String** — wire `Text` into **In String**, set `Print to Screen = true`, `Print to Log = true`, `Duration = 10`. The `Generate` node streams by default, so this fires once per chunk with `Text` holding only the new text. The last call is an empty terminator: `Text` is empty and `bIsFinal` is `true`. `Node Name` names the node that produced the chunk (`answer` here).
1. **On Turn Complete (Status, DebugInfo, TokensGenerated)** → another **Print String** that prints `"Done."` (or **Enum to String** on `Status`).

That's the whole graph — no `Connect`, no `Create Session`, no `Create Agent`. Those are the subsystem's job now.

Want the whole reply as one line?

Streaming (`Send = Streamed`) is the `Generate` node's default. To get a single frame with the complete reply instead of per-chunk deltas, either bind **On Answer Full** (fires once at turn end with the full accumulated text, immediately before `OnTurnComplete`) or set the node's **Send** to **Whole** in Step 5.

Driving the session yourself

For full control — a loading screen, or deciding yourself when the session opens — untick **Auto Connect** / **Auto Create Session** and call `Connect` then `Create Session` (or `Create Session From Settings`) yourself. Engines are not a reason to do this: each model's engine comes from the model catalog, and `Create Session` only takes an optional warm-up hint. `Create Agent` still waits for the session to be ready, so the old connect-vs-create race can't happen either way.

______________________________________________________________________

## Step 7 — Test without entering PIE

Before pressing Play, use the **Chat Window** to verify the agent responds — no Blueprint wiring required:

1. Open **Window → Tryll → Tryll Chat**.
1. Select the actor that has your `UTryllAgentComponent` (with `DemoWorkflow` assigned on the component).
1. Click **Start** — the panel launches the server, connects, and creates an editor agent from that component. The status label turns green: **Ready**.
1. Type a message and press **Enter**. You should see a streamed reply.
1. Click **Stop** when done.

See [Test an Agent in the Editor](https://docs.tryllengine.com/v0.10/how-to/test-agent-in-editor.html) for the full walkthrough including troubleshooting.

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

The `Generate` node streams by default (`Send = Streamed`), so each `OnAnswerText` chunk carries a delta. The last call is an empty terminator with `bIsFinal = true` (it prints an empty line), then your `OnTurnComplete` handler prints `Done.`. For a single full-reply frame instead, bind **On Answer Full** or set the node's **Send** to **Whole** (see the tip in Step 6).

Subsequent runs skip the download and load steps — the model stays in the server process, and `TryllStorage/models/` survives across PIE sessions.

______________________________________________________________________

## Where to see what's happening

Open **Window → Output Log** and use the **Categories** filter:

| Category                   | Source                                                                        | Useful for                                                                                                                                                                                                                                                                                                                   |
| -------------------------- | ----------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `LogTryllServer`           | `tryll_server.exe` stdout (drained by the subsystem and re-emitted in Unreal) | `[Listener] Listening on 127.0.0.1:<port>` (in the editor, auto-launch binds loopback on the **Server Port** project setting, default 9100, and falls back to a free port only if that port is taken; Standalone Game and packaged builds always pick a free port), `ConnectionReady`, model load lines, server-side errors. |
| `LogTryll`                 | Plugin (subsystem, agent component, connection thread)                        | Connect attempts, `CreateSession` outcome, agent lifecycle, `DownloadProgress`.                                                                                                                                                                                                                                              |
| `LogBlueprintUserMessages` | `Print String` calls                                                          | The answer text, your "Done." marker.                                                                                                                                                                                                                                                                                        |

The server's output funnels through the Unreal Output Log, so a single window shows everything.

Client Dev writes one log file per run under `%LOCALAPPDATA%\Tryll\projects\<project-key>\logs\`.

______________________________________________________________________

## What you built

- A session connected to a Tryll server that the plugin spawned for you.
- One agent with a one-node graph.
- Model output surfaced to Blueprint as `OnAnswerText` events, which stream token-by-token by default (`Send = Streamed`); set the node's **Send** to **Whole**, or bind **On Answer Full**, for a single full-reply frame.

Everything the plugin exposes in Blueprint — connection, session, agents, models, string storages, tool calls — is listed in the [Blueprint Catalog](https://docs.tryllengine.com/v0.10/reference/client-api/unreal/blueprint-catalog.html).

## Where to go next

- **[Test an Agent in the Editor](https://docs.tryllengine.com/v0.10/how-to/test-agent-in-editor.html)** — iterate on your workflow without entering PIE.
- **[How-to Guides](https://docs.tryllengine.com/v0.10/how-to/index.html)** — RAG, tool calls, guardrails, model management.
- **[Concepts](https://docs.tryllengine.com/v0.10/concepts/index.html)** — the mental model behind the graph abstraction.
- **[Unreal C++ API](https://docs.tryllengine.com/v0.10/reference/client-api/unreal/cpp/index.html)** and the [Blueprint Catalog](https://docs.tryllengine.com/v0.10/reference/client-api/unreal/blueprint-catalog.html).
- **[Auto-launch the Server](https://docs.tryllengine.com/v0.10/how-to/auto-launch-server.html)** — every Unreal-side server-launch knob explained.
- **[Manage models in the editor](https://docs.tryllengine.com/v0.10/how-to/manage-models-in-editor.html)** — download, register, and pin models with explicit progress.

## Troubleshooting

- **"Accept the Tryll Runtime Licence in the Tryll Welcome window first."** — the editor did not start the server because the licence is not accepted. The Tryll Welcome tab opens. Click **Accept licence**.
- **`Tryll server exe not found at '...'` in `LogTryll/Error`** — no variant folder under `<Plugin>/Binaries/ThirdParty/TryllServer/` resolves. The `TryllClient` release zip ships the `Default` folder populated; from the plugin repository's source, unzip the server zip of the same release into it (Step 2).
- **`OnConnectionChanged(false)` fires repeatedly** — the server crashed or didn't start. Check `LogTryllServer` for the failure reason and `%LOCALAPPDATA%\Tryll\projects\<project-key>\logs\` as a fallback.
- **`OnCreateSessionComplete(false)`** — usually a server-side error reading `models.json` or initialising the inference engine. Check `LogTryllServer`.
- **`Default Model Name` dropdown is empty** — register a model first. Open **Window → Tryll → Tryll Model Manager**, download one, and set its **Build registration** to **Experimental** or **Production**; it then appears in the dropdown.
- **`OnError` fires during `CreateAgent`** — read `ErrorMessage`. Common causes: `Default Model Name` is empty or names a model that isn't yet on disk (fails fast — download it via the Model Manager first), or a route targets a node name that doesn't exist.
- **`CreateAgent` takes several seconds even after the model is downloaded** — normal on the first agent using a given model in a session; the server is loading it from disk into RAM/VRAM. Subsequent agents on the same model are instant.
- **Nothing prints after `Send Message`** — bind both **On Answer Text** *and* **On Turn Complete**. With the default `Send = Streamed`, `OnAnswerText` fires once per chunk (each `Text` is a delta), the last call is an empty terminator with `bIsFinal = true`, and `OnTurnComplete` follows. Set the node's **Send** to **Whole** to get a single full-reply frame with the whole text and `bIsFinal = true`.
