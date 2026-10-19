# Choose where models run

Tryll decides two things about every model you load: **which engine runs it** and **where its weights live**. You control the second one; the first is derived for you.

## Engines are derived, not chosen

You no longer pick an engine per model kind, and there is nothing to configure in Project Settings. The model catalog records which engine runs each model, and that is what Tryll uses — always.

In Unity and Unreal this needs no attention at all: register models in **Window ▸ Tryll ▸ Model Manager** (Unity) or the **Tryll Model Manager** panel (Unreal), and everything follows from the catalog.

If you build sessions yourself you *may* name the engines you expect to use, and it is worth doing when a model's backend is slow to start:

```
from tryll_client import InferenceEngine, ModelRole

client.create_session([
    (ModelRole.Language,  InferenceEngine.LlamaCpp),
    (ModelRole.Tts,       InferenceEngine.SherpaOnnx),  # or LlamaCpp for Orpheus / Maya1
])
```

This is a **hint, not a declaration**. Passing nothing is perfectly normal, and loading a model the hint never mentioned works exactly the same. All the hint does is move engine start-up cost to session creation instead of the first use — the difference between a slightly longer loading screen and a pause on a character's first spoken line.

The one load failure in this area is `EngineNotAvailable` (6008): the model needs an engine your **server build** does not include. The message names the model, its role and the engine, and the fix is a server with that engine, not a change to what you sent.

## Placement: RAM or VRAM

Each model can be placed in **system memory (RAM)** or **device memory (VRAM)**, or left on **Auto**.

| Choice             | Meaning                                                                                  |
| ------------------ | ---------------------------------------------------------------------------------------- |
| **Auto** (default) | Device memory when the player's machine has a usable GPU; system memory otherwise.       |
| **RAM**            | Keep the weights in system memory. Slower generation, but leaves VRAM for your renderer. |
| **VRAM**           | Put the weights in device memory.                                                        |

In the editors, select a registered model and use the **Memory placement** control. Or set it per load:

```
from tryll_client import MemoryPlacement

result = client.load_model("Llama 3.1 8B Instruct (Q4_K_M)",
                           placement=MemoryPlacement.Ram)
```

### Why you would move a model to RAM

Your game's renderer and Tryll's language model compete for the same VRAM. A 7B model in VRAM alongside a demanding scene can cost you more in frame time and texture streaming than it gains in tokens per second. Moving the model to RAM trades generation speed for graphics headroom — and on a machine with a small card it may be the only way both fit.

## Placement is a hint, not a guarantee

This matters more than it sounds, because **you are authoring this on your machine and shipping it to someone else's**. A VRAM choice that suits a 24 GB card is not a promise an 8 GB card can keep.

So placement is always a hint. A machine that cannot honour it loads the model the other way and tells you what it did:

```
result = client.load_model("Big Model", placement=MemoryPlacement.Vram)
if result.placement != MemoryPlacement.Vram:
    print(f"Fell back to {result.placement.name} on {result.backend}")
```

Read the result rather than assuming. Three things make it differ from what you asked:

- **The host cannot reach the device.** No GPU backend available, or a CPU-only build of the speech engine.
- **The model is already loaded.** Placement is fixed when a model loads and is deliberately not part of Tryll's model cache — otherwise one model could be resident twice, doubling the memory a VRAM control exists to save. So the first load wins, and a later request reports the *resident* placement. Unload the model first if you need to move it.
- **You asked for Auto.** It always resolves to a concrete RAM or VRAM.

In Unity and Unreal there is nothing to read: a load that did not go where you asked logs a warning naming both placements and the backend, and one that did is quiet. The completion event stays `(model name, success)` on purpose — a model cannot be moved once it is loaded, so there is no decision for your code to make from the value, and widening the event would have broken every existing subscriber and Blueprint for something only a log can use.

## Models with no placement control

**Voice activity detection (VAD)** has no Placement control, and the Model Manager does not show one. A VAD detector is not a model Tryll loads and keeps — it is created inside the speech recognizer for each voice session — so there is nothing for a placement to apply to. It always runs on the CPU, which is almost certainly what you want anyway: it is a very small model processing 32-millisecond windows, and that is the shape that runs *slower* on a GPU because moving the data costs more than the work saved.

## What you cannot choose, and why

**Which GPU API provides VRAM is not a per-model control.** Placement stays RAM / VRAM / Auto. The speech engine (Sherpa / ONNX Runtime) is CPU in the shipped payload; a VRAM hint for those models falls back to RAM.

For llama.cpp, Unity and Unreal *do* pick the GPU API — see below — but they pick it once for the host process, not per model. The wire protocol still never names CUDA or Vulkan.

## Speech models are a special case

**Auto keeps Sherpa text-to-speech on the CPU**, even on a machine with a capable GPU. That is measured, not cautious: for small voice graphs the cost of copying data between host and device outweighs the compute saved, and one shipped Sherpa voice ran roughly five times slower on the GPU than on the CPU.

Override it only if you have measured otherwise on your own target hardware.

llama.cpp SNAC voices (`Orpheus 3B FT (Q4_K_M)`, `Maya1 3B (i1-Q4_K_M)`) are 3B GGUF backbones. They follow the same RAM / VRAM / Auto placement as a language model. The shared SNAC decoder stays on the CPU.

## Which GPU API the server uses

Placement (`auto` / `ram` / `vram`) is **not** a CUDA-versus-Vulkan switch.

**Unity / Unreal (auto-launched server).** Project Settings → Tryll Client → **Llama.cpp backend** (`Vulkan` or `Auto`, default **Vulkan**). The plugin passes `--llama-backend` when it starts `tryll_server.exe`. `Vulkan` is the portable default and a player build skips the CUDA DLLs. `Auto` prefers CUDA on NVIDIA when `ggml-cuda.dll` loads (and a player build copies those DLLs), otherwise Vulkan. Ignored when Auto Launch Server is off.

**Standalone / C++ / Python.** The committed `server-config.json` sets `engines.llama_cpp.backend` to `"vulkan"`. Override per process with `--llama-backend auto|cuda|vulkan|hip|cpu`. The C++ default if the key is absent is `auto`.

The plugin payload ships both llama.cpp plugins plus the llama-cuda redistributables (`cudart` / `cublas`). It does **not** ship cuDNN or a CUDA ONNX Runtime. Players do not need the CUDA Toolkit installed.

## What Auto does not do

Auto is all-or-nothing: a model goes wholly to VRAM or wholly to RAM. It does **not** size the model to fit whatever VRAM happens to be free. A large model on a small card behaves exactly as it did before you set anything — so if you are close to the limit, choose `Ram` deliberately rather than hoping Auto notices.

The Server Monitor **Server** tab shows the same resident answer live: each loaded model lists its resolved placement and the weight bytes currently in RAM versus VRAM. In Unity, [Inspect Server Memory](https://docs.tryllengine.com/v0.9/how-to/inspect-server-memory.html) shows the same placement as device **ours** versus system unattributed (mmap) after you click Refresh.

## Related

- [Server configuration](https://docs.tryllengine.com/v0.9/reference/server-configuration.html) — standalone `engines.llama_cpp.backend` and `--llama-backend`
- [Auto-launch the server](https://docs.tryllengine.com/v0.9/how-to/auto-launch-server.html) — Unity/Unreal project setting
- [Inspect server memory](https://docs.tryllengine.com/v0.9/how-to/inspect-server-memory.html) — live RAM/VRAM after a placement change
- [Wire protocol](https://docs.tryllengine.com/v0.9/reference/wire-protocol.html) — `model_config`, `LoadModelRequest`/`LoadModelResponse`
- [Error codes](https://docs.tryllengine.com/v0.9/reference/error-codes.html) — `InvalidEngineSet` (2005), `EngineNotAvailable` (6008)
