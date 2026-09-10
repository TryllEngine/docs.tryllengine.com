# First Inference in C++

Build the C++ client library from source, run the bundled `test-chat` demo, and see streamed tokens on stdout.

## Before you start

- Visual Studio 2022 with the C++ workload, CMake 3.24+, and Git.
- At least one language model registered in the server's `models.json`.

No need to start the server manually

`tryll_test_chat` discovers and starts `tryll_server.exe` automatically from the build directory. If you prefer to manage the server yourself, pass `--no-managed-server` when launching the demo.

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
test-chat\build\Debug\tryll_test_chat.exe
```

The demo discovers `tryll_server.exe` from the sibling `tryll\build\server\Debug\` directory, starts it, waits for it to be ready, and then connects. Pick a starting workflow with `--workflow demo` or `--workflow rag` if you want something other than the default.

If you want to connect to a server you started yourself (or one on another machine), pass `--no-managed-server`:

```
test-chat\build\Debug\tryll_test_chat.exe --no-managed-server
```

You should see:

```
Connected. Listing models…
  [0] My Local Model (Local)
  [1] Llama-3.2-3B-Instruct (Loaded)
Agent "chat" ready. Type /quit to exit.

You> hello
Assistant> Hi! How can I help you today?
You>
```

Each token of the assistant's reply appears on stdout as it is produced — exactly what your own application will see through the `on_answer_text` callback.

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
    TC::ManagedServerOptions opts;
    opts.exe  = std::filesystem::path{"tryll/build/server/Debug/tryll_server.exe"};
    opts.port = 9100;

    auto session = Tryll::Client::TryllClient::RunAndConnect(opts);
    session.GetClient().CreateSession({ .engine = ::Tryll::InferenceEngine_LlamaCpp });

    // Build the simplest possible graph: one Generate node.
    // default_exit defaults to "" = END, so no wiring needed.
    TC::GraphDescription graph;
    graph.AddGenerate("answer", ::Tryll::NodeParams::GenerateParamsT{})
         .SetStartNode("answer")
         .SetDefaultModelName("My Local Model");

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

Already running a server?

If your server is managed separately (remote host, Docker, etc.) use `Connect` directly — no `ManagedServerOptions` needed:

```
auto client = Tryll::Client::TryllClient::Connect("127.0.0.1", 9100);
client.CreateSession({ .engine = ::Tryll::InferenceEngine_LlamaCpp });
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

- **[How-to Guides](https://docs.tryllengine.com/v0.8/how-to/index.html)** — task recipes for RAG, tool calls, streaming to UI, model setup.
- **[Concepts](https://docs.tryllengine.com/v0.8/concepts/index.html)** — the mental model. Start with [Architecture at a Glance](https://docs.tryllengine.com/v0.8/concepts/architecture-at-a-glance.html).
- **[C++ Client API](https://docs.tryllengine.com/v0.8/reference/client-api/cpp/index.html)** — the header / class surface.

## Troubleshooting

- **`FindSiblingServerExe` error on launch** — the server exe was not found next to the test-chat exe. Rebuild with `cmake --build --preset debug` or pass `--server-exe <path>` to point directly at the server binary. Pass `--no-managed-server` to skip auto-launch entirely and start the server yourself: see [Run the Tryll Server](https://docs.tryllengine.com/v0.8/how-to/run-the-tryll-server.html).
- **`connection refused` with `--no-managed-server`** — the server is not running or is on a different port. Check the server log.
- **Linker errors about `FlatBuffers`** — make sure you are linking the generated target that was built alongside `tryll_client`; the CMake preset handles this automatically.
- **Build picks Ninja accidentally** — pass `--preset default` explicitly or add `-G "Visual Studio 17 2022" -A x64` to the configure command.
