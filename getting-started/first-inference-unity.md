# First Inference in Unity

Unpack a Tryll release, drop the package into your Unity project (the server is bundled inside it), configure one project setting, download a model through the Model Manager, and watch a single `Generate` node print its answer to the Console during Play Mode — no chat UI required.

Working with an AI coding agent?

The Unity package ships an **`AGENTS.md`** at its root. Attach it to your agent's context (Claude Code, Cursor, Copilot, Codex, …) before asking it to write Tryll code — it carries the graph/slot mental model and the ways Tryll differs from a cloud LLM SDK. Agents that fetch docs themselves can start from [`llms.txt`](https://docs.tryllengine.com/llms.txt).

## What you'll build

A scene with a `TryllAgentComponent` on a GameObject that, on `Start`, connects to a locally-spawned Tryll server and creates a one-node agent. You trigger the first inference by right-clicking the component in the Inspector (**Context Menu → Ask Tryll**), and the answer streams into the Unity Console.

## Before you start

- **Unity 6** (6000.x) or later, with a blank 3D project.
- **~5 GB free disk space** for the downloaded model and a working internet connection on first run.
- Familiarity with creating GameObjects, adding components, and writing a short C# MonoBehaviour.

You do **not** need to build anything from source — the distribution ships pre-generated FlatBuffers bindings.

______________________________________________________________________

## Step 1 — Get the distribution

1. Download `tryll-unity-<version>.7z` from the releases page and extract it somewhere temporary (e.g. `C:\Downloads\tryll\`).

1. Inside is the Unity package — **the server is bundled inside it**, so there is nothing else to install:

   ```
   UnityPlugin/                       <-- the UPM package (everything you need)
   ├── package.json
   ├── Runtime/
   ├── Editor/
   └── .Server/Default/               <-- bundled server (rides inside the package)
       ├── tryll_server.exe
       ├── llama.dll, ggml*.dll, ggml-vulkan.dll
       └── data/
           ├── server-config.json
           ├── models.json
           ├── default-canned-responses.txt
           └── default-guardrail-patterns.txt
   ```

The archive ships **no model weights** — you download the models your graph references up front through the Model Manager (Step 4). `CreateAgent` fails fast if a referenced model is not already on disk; the server does **not** download it for you at agent-creation time. Downloaded weights are cached under `.Server/Default/data/.app-data/models/` inside the package.

______________________________________________________________________

## Step 2 — Install the Unity package

Copy the `UnityPlugin/` folder into your project's `Packages/` directory and name it `com.tryll.client`. Unity auto-discovers embedded packages there, so there is **no `manifest.json` edit** to make:

```
<YourProject>/
├── Assets/
├── Packages/
│   ├── manifest.json
│   └── com.tryll.client/        <-- copy UnityPlugin/* here
│       ├── package.json
│       ├── Runtime/
│       ├── Editor/
│       └── .Server/Default/     <-- server rides inside the package
└── <YourProject>.sln
```

Unity imports the package on focus; **Window → Package Manager → In Project** lists **Tryll Client**, and you should see no compile errors.

Keep the package **embedded** here (a writable folder) rather than in a read-only cache — on first run the server downloads models into `.Server/Default/data/.app-data/`, which a read-only package can't allow.

Add this to `.gitignore` to keep the model cache and logs out of source control:

```
/Packages/com.tryll.client/.Server/Default/data/.app-data/
```

Nothing else to copy

The server binary and its `data/` ride **inside** the package's `.Server/Default/` folder, so installing the package installs the server too — one folder to drop in, one to update on a new release.

______________________________________________________________________

## Step 3 — Configure Tryll Client in Project Settings

Open **Edit → Project Settings → Tryll Client** and set:

| Setting                | Value              | Why                                                                                             |
| ---------------------- | ------------------ | ----------------------------------------------------------------------------------------------- |
| **Auto Launch Server** | `true` (default)   | `TryllClient` spawns the bundled server automatically. Uncheck to run a server yourself.        |
| **Llama.cpp backend**  | `Vulkan` (default) | Portable GPU API. Set `Auto` to prefer CUDA on NVIDIA (also ships CUDA DLLs in a player build). |
| **Game Name**          | e.g. `my-game`     | Short slug sent to the server on every session for telemetry grouping.                          |

There is **no server-path setting** — the location is resolved from the package automatically, identically in the Editor and in packaged builds.

Settings are saved to `Assets/Resources/TryllRuntimeSettings.asset` (created on first access).

Models must be registered before use

`CreateAgent` fails fast if a referenced model is not already on disk — the server does not download it for you. Continue to Step 4 to download and register a model via the editor Model Manager before running the graph below.

______________________________________________________________________

## Step 4 — Download and register a model

Model name fields in the Inspector are **dropdown lists** populated from the models you have registered in this project. On a fresh installation those dropdowns are empty — you must open the Model Manager, download a model, and register it before it appears.

1. Open **Window → Tryll → Model Manager**.\
   The window launches the bundled server, connects, and lists every model the server knows about. Wait for the status indicator in the toolbar to show **● Ready**.

1. In the model list, find a language model to start with.\
   **Recommended:** `Gemma 3 4B Instruct (Q4_K_M)` (~3 GB download).\
   **Smaller / quicker:** `Llama 3.2 3B Instruct (Q4_K_M)` (~2 GB download).

1. Click the model row to select it. In the **Model properties** panel on the right:

   - Status shows **Absent** — the files are not on disk yet.
   - Click **Download**. A progress bar appears and the toolbar shows active download count.

1. Once the status changes to **Downloaded**, set the **Build registration** to **Experimental** (usable in the editor, excluded from packaged builds) or **Production** (shipped in builds).

   - For this tutorial, **Experimental** is fine.

1. Close the Model Manager.

The Model Manager writes a `TryllModelManifest` asset to `Assets/Tryll/TryllModelManifest.asset`. Any `Model Name` dropdown in the Inspector now lists the models you registered here.

Registration controls build inclusion

**Experimental** models are available during development but stripped by the build post-processor when you build a player. **Production** models are staged into the `Server/` folder next to the player executable. A model that is downloaded but unregistered (status **None**) is invisible to the build system.

______________________________________________________________________

## Step 5 — Create a workflow asset and set up the scene

1. In the Project window, right-click → **Create → Tryll → Workflow Asset**. Name it `DemoWorkflow`.
1. Select `DemoWorkflow`. In the Inspector, expand **Graph**:
   - Under **Nodes**, click **+** to add one entry.
   - Set **Name** = `answer`.
   - Click the **Params** type picker (shows `<null>` by default) and select **Generate** from the dropdown. The node's parameter fields expand below it — leave them all at their defaults; the **Default Exit** field stays empty, which routes the turn to END.
   - Set **Start Node** = `answer`.
   - Set **Default Model Name** using the dropdown — it now lists the models you registered in the Model Manager. Pick the one you downloaded in Step 4.
1. Create a new empty **GameObject** — call it `TryllDemo`.
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
            if (isDelta)
                Debug.Log($"[Tryll] delta from {nodeName}: {text}");
            else if (isFinal)
                Debug.Log($"[Tryll] final from {nodeName}: {text}");
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

See [Test an Agent in the Editor](https://docs.tryllengine.com/how-to/test-agent-in-editor.html) for the full walkthrough including troubleshooting.

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
1. Watch the Console — answer deltas stream in as `[Tryll] delta: …` lines, followed by `[Tryll] turn complete`.

______________________________________________________________________

## Where to see what's happening

Open the **Console** window and filter by `[Tryll]`:

| Prefix          | Source                                               | Useful for                                                                                                                     |
| --------------- | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `[TryllClient]` | `TryllClient` singleton                              | Connection attempts, session config, agent lifecycle.                                                                          |
| `[TryllAgent]`  | `TryllAgentComponent`                                | Agent created/destroyed, `DownloadProgress`, errors.                                                                           |
| `[TryllServer]` | `tryll_server.exe` stdout (re-emitted by the client) | `[Listener] Listening on 127.0.0.1:<port>` (auto-launch binds loopback on an OS-chosen port), model load lines, server errors. |

A rotating server log is also written next to the bundled server, at `Packages/com.tryll.client/.Server/Default/data/.app-data/logs/tryll.log` (production log mode is the default in the shipped `server-config.json`).

______________________________________________________________________

## What you built

- A session connected to a Tryll server the client spawned for you.
- One agent with a one-node graph authored in the `DemoWorkflow` Workflow Asset (`Generate` node, driven by a model you downloaded and registered in the Model Manager).
- Model output surfaced as `OnAnswerText` events, which stream token-by-token by default (`Send = Streamed` on the `Generate` node); set `Send = Whole` for a single full-reply frame instead of delta chunks.
- A `[ContextMenu]` method as the simplest way to trigger a turn from the Inspector without any UI setup.

______________________________________________________________________

## Where to go next

- **[Test an Agent in the Editor](https://docs.tryllengine.com/how-to/test-agent-in-editor.html)** — iterate on your workflow without entering Play Mode.
- **[How-to Guides](https://docs.tryllengine.com/how-to/index.html)** — RAG, tool calls, guardrails, model management.
- **[Concepts](https://docs.tryllengine.com/concepts/index.html)** — the mental model behind the graph abstraction.
- **[Unity Client API Reference](https://docs.tryllengine.com/reference/client-api/unity/index.html)** — complete class-level reference.
- **[Manage models in the editor](https://docs.tryllengine.com/how-to/manage-models-in-editor.html)** — download, register, and pin models with explicit progress.

______________________________________________________________________

## Troubleshooting

- **Compile errors on import** — check that the package's `Runtime/Generated/` folder contains `.cs` files. If it's empty, the distribution was packaged without the FlatBuffers step; contact support or re-download.
- **`[TryllServer]` server not found** — the package's `.Server/Default/` folder is missing or empty. A release package ships it populated; if you cloned the repo instead, build the Tryll server so the committed `.Server/Default` symlink resolves to your build.
- **Model name dropdown is empty** — open the Model Manager (**Window → Tryll → Model Manager**), download a model, and set its build registration to **Experimental** or **Production**. Unregistered models do not appear in Inspector dropdowns.
- **Stuck at connecting** — the server crashed or didn't start in time. Check the Console for `[TryllServer]` lines and `.Server/Default/data/.app-data/logs/tryll.log` as a fallback.
- **`CreateSession` fails** — usually a server-side error reading `models.json` or initialising the inference engine. Check `[TryllServer]` Console output.
- **`OnError` fires during `CreateAgent`** — read the error message. Common causes: **Default Model Name** is empty or doesn't match any entry in `models.json` (fails fast — download it via the Model Manager first), or a route targets a node that doesn't exist.
- **Right-clicking the script shows no `Ask Tryll` option** — you must be in Play Mode with the agent already created. Check the Console for the `agent ready` log line first.
- **Nothing appears in the Console** — confirm `OnAnswerText` is wired in `Start()` and that the script is attached to the same GameObject as `TryllAgentComponent`. With the default `Send = Streamed` on the `Generate` node, `OnAnswerText` fires once per chunk (`isDelta = true`), the last chunk has `isFinal = true`, and `OnTurnComplete` follows. Set `Send = Whole` to get a single full-reply frame.
