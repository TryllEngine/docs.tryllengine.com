# First Inference in Unity

Install the Tryll package from its Git URL, let the Welcome window download the server, configure one project setting, download a model through the Model Manager, and watch a single `Generate` node print its answer to the Console during Play Mode — no chat UI required.

Working with an AI coding agent?

The Unity package ships an **`AGENTS.md`** at its root. Attach it to your agent's context (Claude Code, Cursor, Copilot, Codex, …) before asking it to write Tryll code — it carries the graph/slot mental model and the ways Tryll differs from a cloud LLM SDK. Agents that fetch docs themselves can start from [`llms.txt`](https://docs.tryllengine.com/v0.10/llms.txt).

## What you'll build

A scene with a `TryllAgentComponent` on a GameObject that, on `Start`, connects to a locally-spawned Tryll server and creates a one-node agent. You trigger the first inference by right-clicking the component in the Inspector (**Context Menu → Ask Tryll**), and the answer streams into the Unity Console.

## Before you start

- **Unity 6.3** (6000.3) or later, with a blank 3D project, on Windows x64.
- **[Git](https://git-scm.com/downloads)** installed and on your `PATH`: the Package Manager uses it to fetch the package.
- **~5 GB free disk space** for the server and the downloaded model, and a working internet connection on first run.
- Familiarity with creating GameObjects, adding components, and writing a short C# MonoBehaviour.

You do **not** need to build anything from source — the distribution ships pre-generated FlatBuffers bindings.

______________________________________________________________________

## Step 1 — Find the version

Open the [Tryll Unity releases](https://github.com/TryllEngine/tryll-unity-plugin/releases) and note the newest version tag, such as `v1.2.3`. The plugin and the server must be the same version; the package pins its server for you.

The package ships **no model weights** — you download the models your graph references up front through the Model Manager (Step 4). `CreateAgent` fails fast if a referenced model is not already on disk; the server does **not** download it for you at agent-creation time. In Client Dev, downloaded weights land under `<Project>/TryllStorage/models/`.

______________________________________________________________________

## Step 2 — Install the Unity package

1. In Unity, open **Window → Package Manager**.

1. Click **+ → Install package from git URL…**.

1. Paste the URL with the version at the end, and click **Install**:

   ```
   https://github.com/TryllEngine/tryll-unity-plugin.git#v<version>
   ```

**Window → Package Manager → In Project** then lists **Tryll Client**, and you should see no compile errors. To update later, install the same URL with the newer version at the end.

```
<YourProject>/
├── Assets/
├── TryllStorage/                    <-- default storage root (outside Assets/)
└── Packages/
    └── manifest.json                <-- "com.tryll.client": "https://github.com/...#v<version>"
```

### Accept the licence and get the server

After the import, the **Welcome to Tryll** window opens. You can also open it from **Window → Tryll → Welcome**. It has two setup steps:

1. **Licence.** Click **Accept licence**. The editor starts the server only after you accept the Tryll Runtime Licence. The licence includes telemetry, which is always on in the editor. The game you ship sends no telemetry. **Telemetry** opens the [Telemetry](https://docs.tryllengine.com/v0.10/reference/telemetry.html) page. **Read the licence** shows the full text from the package's `LICENSE.md`.
1. **Server.** Click **Download server**. The window downloads the server for this package version from the same GitHub release, checks its SHA-256, and installs it once per version for every project on the computer (`%LOCALAPPDATA%\Tryll\server\`). With an NVIDIA GPU you can click **Download CUDA build** instead. No internet? Download the server zip from the release page and use **Install from zip…**.

The setup line reads **2 of 2 done**. **Withdraw** takes the acceptance back and stops the editor's server.

Model downloads and authored storage files live under `<Project>/TryllStorage/`, not inside the package. Add this to `.gitignore` if you do not want models in source control (committing them is optional):

```
/TryllStorage/models/
```

Got a `tryll-unity-<version>.7z` package instead?

Partner packages carry the server inside the package. Copy its `UnityPlugin/` folder to `Packages/com.tryll.client/`; the Welcome window's server step is then done as soon as you accept the licence.

______________________________________________________________________

## Step 3 — Configure Tryll Client in Project Settings

Open **Edit → Project Settings → Tryll Client** and set:

| Setting                 | Value                    | Why                                                                                                                                                                                                                                                 |
| ----------------------- | ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Auto Launch Server**  | `true` (default)         | `TryllClient` spawns the server automatically. Uncheck to run a server yourself.                                                                                                                                                                    |
| **Llama.cpp backend**   | `Vulkan` (default)       | Portable GPU API. `Auto` prefers CUDA on NVIDIA and ships the CUDA DLLs in a player build; it needs the CUDA server, so first click **Download CUDA build** in the Welcome window (the default download has no CUDA and a player build then fails). |
| **Game Name**           | e.g. `my-game`           | Human project label. Sent as the session name and as `--project-name`. Telemetry groups by a hidden project id; see [Telemetry](https://docs.tryllengine.com/v0.10/reference/telemetry.html).                                                       |
| **Telemetry Level**     | `Basic` (default)        | `Turns` or `Full` send more while debugging. Takes effect the next time the editor launches the server.                                                                                                                                             |
| **Storage Data Folder** | `TryllStorage` (default) | Project folder **outside** `Assets/`. The editor launches the server with `--storage-root` here.                                                                                                                                                    |

There is **no server-path setting**. The editor finds the server for this package version itself (the one the Welcome window downloaded, or the one inside a partner package), and a player build copies it next to the game. The editor sends content-free telemetry at the `basic` level. To send turn timings or conversation text while debugging, see [Telemetry](https://docs.tryllengine.com/v0.10/reference/telemetry.html). Packaged players send nothing.

Settings are saved to `Assets/Resources/TryllRuntimeSettings.asset` (created on first access).

Models must be registered before use

`CreateAgent` fails fast if a referenced model is not already on disk — the server does not download it for you. Continue to Step 4 to download and register a model via the editor Model Manager before running the graph below.

______________________________________________________________________

## Step 4 — Download and register a model

Model name fields in the Inspector are **dropdown lists** populated from the models you have registered in this project. On a fresh installation those dropdowns are empty — you must open the Model Manager, download a model, and register it before it appears.

1. Open **Window → Tryll → Model Manager**.\
   The window launches the server, connects, and lists every model the server knows about. Wait for the status indicator in the toolbar to show **● Ready**.

1. In the model list, find a language model to start with.\
   **Recommended:** `Gemma 4 E4B Instruct (Q4_K_M)` (~5.4 GB download, about 6 GB of VRAM). The demo workflows use it.\
   **Smaller / quicker:** `Llama 3.2 3B Instruct (Q4_K_M)` (~2 GB download, about 2.6 GB of VRAM). Pick this one on a 6 GB card.

1. Click the model row to select it. In the **Model properties** panel on the right:

   - Status shows **Absent** — the files are not on disk yet.
   - Click **Download**. A progress bar appears and the toolbar shows active download count.

1. Once the status changes to **Downloaded**, set the **Build registration** to **Experimental** (usable in the editor, excluded from packaged builds) or **Production** (shipped in builds).

   - For this tutorial, **Experimental** is fine.

1. Close the Model Manager.

The Model Manager writes a `TryllModelManifest` asset to `Assets/Resources/TryllModelManifest.asset`. Any `Model Name` dropdown in the Inspector now lists the models you registered here.

Registration controls build inclusion

**Experimental** models are available during development but stripped by the build post-processor when you build a player. **Production** models are staged into the `Server/` folder next to the player executable. A model that is downloaded but unregistered (status **None**) is invisible to the build system.

______________________________________________________________________

## Step 5 — Create a workflow asset and set up the scene

1. In the Project window, right-click → **Create → Tryll → Workflow Asset**. Name it `DemoWorkflow`.

   The **Create → Tryll** submenu (the same menu as **Assets → Create → Tryll**).

   2. Select `DemoWorkflow`. In the Inspector, expand **Graph**:

   - Under **Nodes**, click **+** to add one entry.
   - Set **Name** = `answer`.
   - Click the **Params** type picker (shows `<null>` by default) and select **Generate** from the dropdown. The node's parameter fields expand below it — leave them all at their defaults; the **Default Exit** field stays empty, which routes the turn to END.
   - Set **Start Node** = `answer`.
   - Set **Default Model Name** using the dropdown — it now lists the models you registered in the Model Manager. Pick the one you downloaded in Step 4.

   3. Create a new empty **GameObject** — call it `TryllDemo`.
   1. Add a **Tryll Agent Component** (`Add Component → Tryll → Tryll Agent Component`).
   1. Assign `DemoWorkflow` to the component's **Workflow Asset** field.
   1. **Un-check Auto Create On Connect** — this tutorial wires the lifecycle explicitly so you control exactly when the agent is created.
   1. Create a C# script `TryllDemo.cs` and attach it to the same GameObject:

```
using UnityEngine;
using Tryll.Client;

public class TryllDemo : MonoBehaviour
{
    private TryllAgentComponent _agent;

    void Start()
    {
        _agent = GetComponent<TryllAgentComponent>();

        _agent.OnAnswerText.AddListener((nodeName, text, isDelta, isFinal) =>
        {
            // Send = Streamed: every chunk is a delta, and the last one is an
            // empty terminator with isFinal = true.
            // Send = Whole: one chunk with isDelta = false and the full text.
            if (!string.IsNullOrEmpty(text))
                Debug.Log($"[Tryll] {(isDelta ? "delta" : "reply")} from {nodeName}: {text}");
            if (isFinal)
                Debug.Log($"[Tryll] {nodeName} finished its answer");
        });

        _agent.OnTurnComplete.AddListener((status, debugInfo, tokens) =>
            Debug.Log($"[Tryll] turn complete — status={status} tokens={tokens}"));

        _agent.OnError.AddListener(error =>
            Debug.LogError($"[Tryll] error: {error}"));

        _agent.OnAgentCreated.AddListener(() =>
            Debug.Log("[Tryll] agent ready — right-click this component and choose 'Ask Tryll'"));

        var client = TryllClient.Instance;
        client.CreateSessionComplete += err =>
        {
            if (!err.IsOk) { Debug.LogError($"[Tryll] CreateSession failed: {err}"); return; }
            _agent.CreateAgent();
        };

        client.StartSession();
    }

    [ContextMenu("Ask Tryll")]
    void AskTryll()
    {
        if (!_agent.HasAgent)
        {
            Debug.LogWarning("[Tryll] agent not ready yet");
            return;
        }
        _agent.SendMessage("In one sentence: what is Tryll?");
    }
}
```

______________________________________________________________________

## Step 6 — Test without entering Play Mode

Before pressing **Play**, use the **Chat Window** to verify your setup is working:

1. Open **Window → Tryll → Chat**.
1. Confirm the **Tryll Settings** field shows your `TryllRuntimeSettings` asset (auto-found).
1. Drag the `TryllDemo` GameObject's **Tryll Agent Component** into the **Agent Component** slot.
1. Click **Start** — the window launches the server, connects, and creates the agent. The status label turns green: **Ready**.
1. Type a message and press **Enter**. You should see a streamed reply.
1. Click **Stop** when done.

See [Test an Agent in the Editor](https://docs.tryllengine.com/v0.10/how-to/test-agent-in-editor.html) for the full walkthrough including troubleshooting.

______________________________________________________________________

## Step 7 — Enter Play Mode

Press **Play**.

On the **first** run you'll see:

1. A brief pause while the server starts and the client connects.
1. The model loads from disk (a few seconds the first time it is used in a session) — this only works because you already downloaded and registered it in Step 4; `CreateAgent` fails fast otherwise.
1. The Console prints: `[Tryll] agent ready — right-click this component and choose 'Ask Tryll'`.

Once the agent is ready:

1. In the **Hierarchy**, select the `TryllDemo` GameObject.
1. In the **Inspector**, right-click the `TryllDemo` script component header and choose **Ask Tryll**.
1. Watch the Console — answer deltas stream in as `[Tryll] delta from answer: …` lines, then `[Tryll] answer finished its answer` and `[Tryll] turn complete`.

______________________________________________________________________

## Where to see what's happening

Open the **Console** window and filter by `[Tryll]`:

| Prefix          | Source                                               | Useful for                                                                                                                                                                                                                                                                     |
| --------------- | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `[TryllClient]` | `TryllClient` singleton                              | Connection attempts, session config, agent lifecycle.                                                                                                                                                                                                                          |
| `[TryllAgent]`  | `TryllAgentComponent`                                | Agent created/destroyed, `DownloadProgress`, errors.                                                                                                                                                                                                                           |
| `[TryllServer]` | `tryll_server.exe` stdout (re-emitted by the client) | `[Listener] Listening on 127.0.0.1:<port>` (in the editor, auto-launch binds loopback on the **Server Port** project setting, default 9100, and falls back to a free port only if that port is taken; player builds always pick a free port), model load lines, server errors. |

Client Dev writes one log file per run under `%LOCALAPPDATA%\Tryll\projects\<project-key>\logs\` (project key from the sanitized game name plus a hash of the project directory).

______________________________________________________________________

## What you built

- A session connected to a Tryll server the client spawned for you.
- One agent with a one-node graph authored in the `DemoWorkflow` Workflow Asset (`Generate` node, driven by a model you downloaded and registered in the Model Manager).
- Model output surfaced as `OnAnswerText` events, which stream token-by-token by default (`Send = Streamed` on the `Generate` node); set `Send = Whole` for a single full-reply frame instead of delta chunks.
- A `[ContextMenu]` method as the simplest way to trigger a turn from the Inspector without any UI setup.

______________________________________________________________________

## Where to go next

- **[Test an Agent in the Editor](https://docs.tryllengine.com/v0.10/how-to/test-agent-in-editor.html)** — iterate on your workflow without entering Play Mode.
- **[How-to Guides](https://docs.tryllengine.com/v0.10/how-to/index.html)** — RAG, tool calls, guardrails, model management.
- **[Concepts](https://docs.tryllengine.com/v0.10/concepts/index.html)** — the mental model behind the graph abstraction.
- **[Unity Client API Reference](https://docs.tryllengine.com/v0.10/reference/client-api/unity/index.html)** — complete class-level reference.
- **[Manage models in the editor](https://docs.tryllengine.com/v0.10/how-to/manage-models-in-editor.html)** — download, register, and pin models with explicit progress.

______________________________________________________________________

## Troubleshooting

- **Compile errors on import** — check that the package's `Runtime/Generated/` folder contains `.cs` files. If it's empty, the distribution was packaged without the FlatBuffers step; contact support or re-download.
- **"Accept the Tryll Runtime Licence in the Tryll Welcome window first."** — the editor did not start the server because the licence is not accepted. The Welcome window opens. Click **Accept licence**, then try again (for example, click **Start** again in the Chat window).
- **`[TryllServer]` server not found** — the server for this package version is not installed. Open **Window → Tryll → Welcome** and click **Download server**, or **Install from zip…** with the server zip from the release page. (A partner `.7z` package carries it in `.Server/Default/`.)
- **Model name dropdown is empty** — open the Model Manager (**Window → Tryll → Model Manager**), download a model, and set its build registration to **Experimental** or **Production**. Unregistered models do not appear in Inspector dropdowns.
- **Stuck at connecting** — the server crashed or didn't start in time. Check the Console for `[TryllServer]` lines and the Client Dev log under `%LOCALAPPDATA%\Tryll\projects\<project-key>\logs\` as a fallback.
- **`CreateSession` fails** — usually a server-side error reading `models.json` or initialising the inference engine. Check `[TryllServer]` Console output.
- **`OnError` fires during `CreateAgent`** — read the error message. Common causes: **Default Model Name** is empty or doesn't match any entry in `models.json` (fails fast — download it via the Model Manager first), or a route targets a node that doesn't exist.
- **Right-clicking the script shows no `Ask Tryll` option** — you must be in Play Mode with the agent already created. Check the Console for the `agent ready` log line first.
- **Nothing appears in the Console** — confirm `OnAnswerText` is wired in `Start()` and that the script is attached to the same GameObject as `TryllAgentComponent`. With the default `Send = Streamed` on the `Generate` node, `OnAnswerText` fires once per chunk (`isDelta = true`), the last call is an empty terminator (`text` is empty, `isFinal = true`), and `OnTurnComplete` follows. Set `Send = Whole` to get a single full-reply frame (`isDelta = false`, `isFinal = true`).
