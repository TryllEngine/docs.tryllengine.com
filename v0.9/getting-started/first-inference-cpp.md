# First Inference in C++

Build the C++ client library from source, run the bundled `test-chat` demo, and see streamed tokens on stdout.

## Before you start

- Visual Studio 2022 with the C++ workload, CMake 3.24+, and Git.
- An internet connection the first time you run the demo. It downloads `Llama 3.2 3B Instruct (Q4_K_M)` from the server's `models.json` catalog if that model is not on disk yet.

No need to start the server manually

`tryll_test_chat` finds `tryll_server.exe` in the `tryll\build` tree and starts it automatically. If you prefer to manage the server yourself, launch the demo with `--profile external-text`.

## Step 1 — build the two projects

The server and the console demo are **two separate CMake projects**. Build the server first (its tree also produces the `tryll_client` static library), then build `test-chat`:

```
git clone <tryll-repo-url>

:: 1. Server + C++ client library — built from the tryll/ tree.
cd tryll
cmake --preset default
cmake --build --preset debug

:: 2. test-chat console demo — its own CMake project that links tryll/
::    with the server disabled.
cd ..\test-chat
cmake --preset default
cmake --build --preset debug
```

The `default` preset uses the Visual Studio 17 2022 generator with CUDA and Vulkan `PREBUILT` — no GPU SDK required. For a CPU-only build add `-DTRYLL_VULKAN=OFF -DTRYLL_CUDA=OFF`. The first build produces `tryll_server.exe` under `tryll\build\server\Debug\`; the second produces `tryll_test_chat.exe` under `test-chat\build\Debug\`.

One-shot build-and-run

`test-chat\build_and_run_dbg.bat` runs both builds in order and then launches the demo — handy while iterating.

## Step 2 — run the demo

```
test-chat\build\Debug\tryll_test_chat.exe --profile managed-text
```

Profiles live in `test-chat\data\test-chat-config.json`. The default profile, `managed-voice`, turns on speech output and push-to-talk voice input, so this tutorial picks `managed-text`: a typed chat on the `demo` workflow. Add `--workflow rag` or `--workflow tools` to start on a different workflow.

The demo looks for `tryll_server.exe` under `tryll\build\server\` (the `Debug`, `Release`, then `Production` folder), starts it, waits for it to be ready, and then connects. Set `TRYLL_SERVER_EXE` or pass `--server-exe <path>` to use a different binary.

If you want to connect to a server you started yourself on `127.0.0.1:9100`, use the `external-text` profile instead. Add `--server-host` and `--server-port` for another address:

```
test-chat\build\Debug\tryll_test_chat.exe --profile external-text
```

After the launch lines, the demo prints the session id and a table of the models the server lists, then the prompt:

```
[test-chat] Session <id> ready
[test-chat] Models returned by the server for this session's engine (…)
[test-chat] ListModels count: …
…
[test-chat] Workflow: demo. Type a message or /help. Press Ctrl+C or Ctrl+Z then Enter to quit.

You: hello
AI (generate): Hi! How can I help you today?
You:
```

Each token of the reply appears on stdout as it is produced, which is exactly what your own application will see through the `SetOnAnswerText` callback. The name in brackets is the node that produced the text. Type `/help` to list the demo's commands.

## Step 3 — write your own minimal client

For a smaller starting point than `test-chat`, use `RunAndConnect` to spawn the server and connect in a single call. The returned `ConnectedSession` owns both the child process and the TCP session; its destructor shuts them down in the correct order:

```
#include <filesystem>
#include <future>
#include <iostream>
#include <string_view>

#include <tryll/TryllClient.h>

int main()
{
    namespace TC = Tryll::Client;

    // Spawn the server and connect. Throws Tryll::Client::TryllError on failure.
    // Leaving opts.port at its default (9100) picks a free port.
    TC::ManagedServerOptions opts;
    opts.exe = std::filesystem::path{"tryll/build/server/Debug/tryll_server.exe"};

    auto session = Tryll::Client::TryllClient::RunAndConnect(opts);
    session.GetClient().CreateSession({});

    // Build the simplest possible graph: one Generate node.
    // default_exit defaults to "" = END, so no wiring needed.
    TC::GraphDescription graph;
    graph.AddGenerate("answer", ::Tryll::NodeParams::GenerateParamsT{})
         .SetStartNode("answer")
         .SetDefaultModelName("Llama 3.2 3B Instruct (Q4_K_M)");   // downloaded by test-chat

    auto agent = session.GetClient().CreateAgent(graph);

    // SendText is fire-and-forget, so we must block until the turn finishes;
    // otherwise the session (and the process) would tear down before the
    // streamed tokens arrive. Signal completion from OnTurnComplete.
    std::promise<void> turnDone;
    agent.SetOnAnswerText([](std::string_view /*nodeName*/, std::string_view text,
                             bool /*isDelta*/, bool /*isFinal*/)
    {
        std::cout << text << std::flush;   // streams by default (SendAnswer::Streamed)
    });
    agent.SetOnTurnComplete([&turnDone](::Tryll::TurnStatus, std::string_view, std::int32_t)
    {
        std::cout << "\n";
        turnDone.set_value();
    });

    agent.SendText("In one sentence: what is Tryll?");
    turnDone.get_future().wait();   // block until OnTurnComplete fires

    agent.Destroy();
    // session destructor: shuts down the client, then stops the server
}
```

`ManagedServerOptions::port` defaults to 9100, and that default means "pick a free port": the launcher allocates an unused port and starts the server on it. Only a different value is bound exactly as given. To connect a second client to the same server, use the port it really bound, `session.GetServer().Port()`.

Already running a server?

If your server is managed separately (remote host, Docker, etc.) use `Connect` directly — no `ManagedServerOptions` needed. A server you start yourself listens on the `port` in its `server-config.json` (9100 in the shipped file):

```
auto client = Tryll::Client::TryllClient::Connect("127.0.0.1", 9100);
client.CreateSession({});
// …
client.Shutdown();
```

Link it against `tryll_client` and `tryll_common` (both already built by the preset):

```
target_link_libraries(my_app PRIVATE tryll_client tryll_common)
```

## What you built

- A C++ client that connects, creates the session, creates a one-node agent, and streams a reply to stdout.
- The same surface as the bundled `test-chat` demo, which is a good reference when you are ready to add RAG, tool calls, or guardrails.

## Where to go next

- **[How-to Guides](https://docs.tryllengine.com/v0.9/how-to/index.html)** — task recipes for RAG, tool calls, streaming to UI, model setup.
- **[Concepts](https://docs.tryllengine.com/v0.9/concepts/index.html)** — the mental model. Start with [Tryll in Ten Minutes](https://docs.tryllengine.com/v0.9/concepts/tryll-in-ten-minutes.html).
- **[C++ Client API](https://docs.tryllengine.com/v0.9/reference/client-api/cpp/index.html)** — the header / class surface.

## Troubleshooting

- **`FindServerExe: could not locate tryll_server`** — the server exe was not found under `tryll\build\server\Debug`, `Release`, or `Production` in the repository the demo was built from. Build the server with `cmake --build --preset debug` in `tryll\`, set `TRYLL_SERVER_EXE`, or pass `--server-exe <path>` to point directly at the server binary. Use `--profile external-text` to skip auto-launch entirely and start the server yourself: see [Run the Tryll Server](https://docs.tryllengine.com/v0.9/how-to/run-the-tryll-server.html).
- **`connection refused` with `--profile external-text`** — the server is not running or is on a different port. Check the server log, and pass `--server-port` if it does not listen on 9100.
- **Linker errors about `FlatBuffers`** — make sure you are linking the generated target that was built alongside `tryll_client`; the CMake preset handles this automatically.
- **Build picks Ninja accidentally** — pass `--preset default` explicitly or add `-G "Visual Studio 17 2022" -A x64` to the configure command.
