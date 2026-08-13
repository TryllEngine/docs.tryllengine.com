# Use Your Own Local Model

Register a GGUF file already on disk so Tryll can load it without a Hugging Face download.

Temporary workflow

Registering a local model currently requires editing `data/models.json` at server startup. In the future there would be implemented dedicated RegisterModel() client API for handling this.

Prerequisites

- A `.gguf` file on the server machine, e.g. `C:\models\my-model-q4_k_m.gguf`.
- Write access to the server's `data/models.json`.
- The server configured for `LlamaCpp` — see [Connect and Manage a Session](https://docs.tryllengine.com/how-to/connect-and-manage-session.html).

## Steps

1. **Add an entry to `data/models.json`** with a `local_path` field instead of (or in addition to) Hugging Face source fields. The key fields are `name` plus one `variants` entry with `engine = "llama-cpp"`, a `local_path` **directory**, and the `files` inside it:

   ```
   {
     "name": "My Local Model",
     "variants": [
       {
         "engine": "llama-cpp",
         "local_path": "C:/models",
         "files": ["my-model-q4_k_m.gguf"],
         "kv_cache_type": "q8_0"
       }
     ]
   }
   ```

   Notes:

   - `name` is how clients will refer to the model. Choose something short and stable.
   - Language models carry **no** `model_type` field — it is omitted and language is the implicit default. (Only non-language entries set `model_type`, e.g. `"embedding"`, `"stt"`, `"tts"`, `"vad"`.)
   - `local_path` is a **directory** (not the file itself), absolute or relative to the server executable's working directory; `files` lists the GGUF file name(s) inside it. Forward slashes are fine on Windows. Omit `local_path` and use `huggingface_repo` instead to have the server download the files.
   - `kv_cache_type` is optional. Default is `q8_0`; use `f16` for maximum fidelity or `q4_0` to halve KV memory on large contexts.
   - There is **no** `tool_call_format` field. Tool calling is driven by the model's own chat template — see [Tool Calling](https://docs.tryllengine.com/concepts/tool-calling.html).

   For the full schema see [Model Management](https://docs.tryllengine.com/reference/model-management.html).

1. **Restart the server.** `models.json` is read at startup; Tryll does not currently watch the file. The first server log line after restart should list your model alongside the existing catalog entries.

1. **Verify via `ListModels`.** From a client:

   ```
   var (models, error) = await TryllClient.Instance.RequestListModelsAsync();
   if (!error.IsOk) { Debug.LogError(error.Message); return; }
   foreach (var m in models)
       Debug.Log($"{m.Name} {m.Status}");   // expect "My Local Model Local"
   ```

   Call `UTryllSubsystem::ListModels` (Blueprint: **Tryll|Models → List Models**). Bind **On List Models Complete** to iterate `TArray<FTryllModelInfo>`.

   ```
   auto models = client.ListModels();
   for (const auto& m : models) {
       std::cout << m.name << " "
                 << static_cast<int>(m.status) << "\n";
   }
   ```

   ```
   for m in client.list_models():
       print(m.name, m.status.name)   # expect "My Local Model Local"
   ```

   Your new entry should appear with `ModelStatus.Local`, meaning the file is present on disk and can be loaded. If you see `Absent`, Tryll could not find the file — re-check `local_path`.

1. **Reference the model from a graph.** Pass the `name` you chose as `model_name` on any `Generate` or `ToolCall` node, or as the agent's `default_model_name`:

   ```
   from tryll_client.graph import GraphDescription, GenerateParams

   graph = (
       GraphDescription()
       .add_node("answer", GenerateParams(model_name="My Local Model"))
   )
   # or, more commonly, set it as the graph's default:
   graph.set_default_model_name("My Local Model")
   ```

1. **(Optional) pin it.** Load the model eagerly so the first turn does not pay the load cost:

   ```
   client.load_model("My Local Model")
   ```

   See [Pin and Unpin Models](https://docs.tryllengine.com/how-to/pin-and-unpin-models.html) for the trade-offs.

## Verify it worked

Create an agent using the model and send one message. A successful load shows up server-side as:

```
[info] Loading model "My Local Model" from C:/models/my-model-q4_k_m.gguf
[info] Model loaded: ctx=4096 vocab=128256 dtype=q8_0
```

## Common pitfalls

- **`Absent` status.** `local_path` is wrong or the file is not readable by the server process. Try opening it as the same user account that runs the server.
- **Model loads but refuses prompts / produces garbage.** Usually the chat template embedded in the GGUF is missing or wrong for the model. Tryll renders prompts through the model's own chat template (see [Tool Calling](https://docs.tryllengine.com/concepts/tool-calling.html) for how tool calls ride on it), so re-export the GGUF with a correct template, or pick a different quantisation of the same weights.
- **Out-of-memory on load.** Either use a smaller quant (e.g. Q4_K_M instead of Q8_0), switch `kv_cache_type` to `q4_0`, or use a smaller context window at `CreateSession` time (not yet user-facing — currently fixed at model default).

## Related

- [Reference: Model Management](https://docs.tryllengine.com/reference/model-management.html)
- [Reference: Server Configuration](https://docs.tryllengine.com/reference/server-configuration.html)
- [Concept: Models and Inference Engines](https://docs.tryllengine.com/concepts/models-and-inference-engines.html)
- [How-to: Pin and Unpin Models](https://docs.tryllengine.com/how-to/pin-and-unpin-models.html)
