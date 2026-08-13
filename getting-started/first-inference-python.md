# First Inference in Python

Install the Python client, connect to a running Tryll server, and stream a reply from a local language model — all in about twenty lines of code.

## Before you start

- The Tryll server is running on `127.0.0.1:9100`. If not, see [Run the Tryll Server](https://docs.tryllengine.com/how-to/run-the-tryll-server.html).
- At least one language model is listed in the server's `data/models.json` with status `Local` or `Loaded`. You can check with any client, or bring your own — see [Use Your Own Local Model](https://docs.tryllengine.com/how-to/use-your-own-local-model.html).
- Python 3.10+.

## Step 1 — install the client

```
pip install tryll-client
```

That installs the `tryll_client` package with no external Python dependencies beyond the standard library.

## Step 2 — write the script

Save this as `first-inference.py`:

```
from tryll_client import TryllClient, InferenceEngine
from tryll_client.graph import GraphDescription, GenerateParams

# Connect and create the session.
client = TryllClient.connect("127.0.0.1", 9100)
client.create_session(InferenceEngine.LlamaCpp)

# Pick a model — use the actual name from your server's models.json.
MODEL = "My Local Model"

# Build the simplest possible graph: one Generate node.
# default_exit defaults to "" = END, so no wiring needed.
graph = (
    GraphDescription()
    .add_node("answer", GenerateParams())
    .set_start_node("answer")
    .set_default_model_name(MODEL)
)

agent = client.create_agent(graph)

# Ask a question and print the full reply.
reply = agent.send_message("In one sentence: what is Tryll?")
print(reply)

agent.destroy()
client.shutdown()
```

## Step 3 — run it

```
python first-inference.py
```

Want the client to start the server for you?

Use `TryllClient.run_and_connect` — it spawns the server, waits for it to be ready, and returns a `ConnectedSession` that shuts everything down automatically:

```
from pathlib import Path
from tryll_client import TryllClient, InferenceEngine
from tryll_client.graph import GraphDescription, GenerateParams

with TryllClient.run_and_connect(
    exe=Path("path/to/tryll_server.exe"),
    port=9100,
) as session:
    session.client.create_session(InferenceEngine.LlamaCpp)

    graph = (
        GraphDescription()
        .add_node("answer", GenerateParams())
        .set_start_node("answer")
        .set_default_model_name("My Local Model")
    )
    agent = session.client.create_agent(graph)

    reply = agent.send_message("In one sentence: what is Tryll?")
    print(reply)
# server is stopped automatically when the `with` block exits
```

For lower-level control, or if you need to manage the server lifetime independently, see [Auto-launch the Server](https://docs.tryllengine.com/how-to/auto-launch-server.html).

Expected output (your model's wording will vary):

```
Tryll is a local small-language-model inference server with C++, Python, and Unreal clients for building on-device AI agents.
```

`send_message` blocks until the server's `TurnComplete` arrives and returns the full reply as a single string — the simplest thing for a script. The server streams tokens as they're generated, and **Python receives them too**: register `agent.set_on_answer_text(cb)` — callback `(node_name, text, is_delta, is_final)` — before sending, or use the non-blocking `agent.send_message_async(...)`. Every client (Python, C++, Unity, Unreal) exposes the same per-token streaming callback; see [Stream Answers to a UI](https://docs.tryllengine.com/how-to/stream-answers-to-ui.html).

## What you built

- A minimal session with one agent.
- A one-node graph: `Generate → END`.
- A blocking `send_message` call that returns the full reply.

The graph is deliberately minimal; everything else in Tryll (retrieval, tool calls, guardrails) plugs into exactly this shape by inserting extra nodes before `Generate`.

## Where to go next

- **[How-to Guides](https://docs.tryllengine.com/how-to/index.html)** — ready-made recipes: RAG, tool calls, streaming to UI, guardrails.
- **[Concepts](https://docs.tryllengine.com/concepts/index.html)** — the mental model. Start with [Architecture at a Glance](https://docs.tryllengine.com/concepts/architecture-at-a-glance.html).
- **[Python Client API](https://docs.tryllengine.com/reference/client-api/python/index.html)** — surface map of `tryll_client`.

## Troubleshooting

- **`ConnectionRefusedError`** — the server is not running on the expected host/port, or a firewall blocked it. Re-check [Run the Tryll Server](https://docs.tryllengine.com/how-to/run-the-tryll-server.html).
- **`TryllError: 4002`** — the model name in your script does not match any entry in `models.json`. List models with `client.list_models()`.
- **Script hangs after `send_message`** — the model is loading for the first time. Watch the server log; the next turn will be fast.
