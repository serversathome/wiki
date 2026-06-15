---
title: Ollama
description: A guide to deploying Ollama
published: true
date: 2026-06-15T15:19:33.594Z
tags: 
editor: markdown
dateCreated: 2026-06-15T14:56:05.778Z
---

# <img src="/ollama.png" class="tab-icon"> What is Ollama?

**Ollama** is a self-hosted runtime for running large language models locally. It pulls quantized open-weight models (Llama 3.x, Mistral, Gemma, Qwen, Phi, DeepSeek, and many more) and serves them over a simple REST API on port `11434`, with a built-in CLI for pulling and chatting with models. It handles model storage, GPU offload, and concurrency for you, so you get a local, private, OpenAI-style endpoint without sending a single token to the cloud.

Ollama has no web interface of its own — it's an API and CLI. Pair it with **Open WebUI** (covered in section 3) for a ChatGPT-style frontend.

> 
> Ollama runs models on CPU by default, but performance is dramatically better with a GPU. NVIDIA cards need the **NVIDIA Container Toolkit** installed on the host before the container can see the GPU.
{.is-info}

# 1 · Deploy Ollama
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

```yaml
services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    ports:
      - "11434:11434"
    volumes:
      - /mnt/tank/configs/ollama:/root/.ollama
    environment:
      - OLLAMA_HOST=0.0.0.0:11434
    restart: unless-stopped
    # --- NVIDIA GPU (requires NVIDIA Container Toolkit on host) ---
    # Uncomment the block below to enable GPU acceleration
    # deploy:
    #   resources:
    #     reservations:
    #       devices:
    #         - driver: nvidia
    #           count: all
    #           capabilities: [gpu]
```

1. The model store lives at `/root/.ollama` inside the container — keep it on persistent storage or every restart re-downloads your models.
2. The API listens on `http://<host-ip>:11434`. Test it once the container is up:
   ```bash
   curl http://localhost:11434/api/tags
   ```
3. Pull and run your first model:
   ```bash
   docker exec -it ollama ollama pull llama3.2
   docker exec -it ollama ollama run llama3.2 "Say hello in one sentence"
   ```

> 
> The volume mount is the one thing you cannot skip. Models are multiple GB each — without persistence you re-download them on every container recreate.
{.is-warning}

## <img src="/truenas.png" class="tab-icon"> TrueNAS

Ollama is available as a **Community** train app in the TrueNAS catalog.

1. Navigate to **Apps** in the TrueNAS UI
2. Click **Discover Apps** and search for **"Ollama"**
3. Click **Install**
4. Configure the following:
   - **Web Port**: `11434` (or your preferred host port)
   - **Storage** → **Models Storage**: set to a **Host Path** of `/mnt/tank/configs/ollama` so models persist on your pool
   - **Resources** → enable **GPU Passthrough** and select your NVIDIA device if you have one
5. Click **Install** / **Save**

> 
> Requires TrueNAS **24.10.2.2 (Electric Eel)** or newer. The catalog app maps the internal `/root/.ollama` model directory for you — point its host path at your `tank` pool, not the default ix-applications dataset, if you want easy backups.
{.is-info}

# 2 · Configuration

## 2.1 Managing Models

Everything is driven through the CLI inside the container (or `ollama` on a host install):

| Command | Action |
|---------|--------|
| `ollama pull <model>` | Download a model |
| `ollama run <model>` | Pull (if needed) and start an interactive chat |
| `ollama list` | List installed models |
| `ollama ps` | Show currently loaded models and VRAM/RAM usage |
| `ollama rm <model>` | Delete a model |
{.dense}

Browse the full catalog of available models at [ollama.com/library](https://ollama.com/library). Watch the parameter size against your hardware — a rough guide is that a model needs slightly more than its file size in RAM or VRAM to run comfortably.

## 2.2 Useful Environment Variables

| Variable | Purpose |
|----------|---------|
| `OLLAMA_HOST` | Bind address/port. The image defaults to `0.0.0.0:11434` so it's reachable through published ports. |
| `OLLAMA_MODELS` | Override the model storage directory (defaults to `/root/.ollama/models`). |
| `OLLAMA_KEEP_ALIVE` | How long a model stays loaded in memory after the last request (e.g. `5m`, `-1` for always). |
| `OLLAMA_NUM_PARALLEL` | Number of parallel requests served per model. |
| `OLLAMA_MAX_LOADED_MODELS` | How many distinct models can be resident in memory at once. |
{.dense}

## 2.3 Using the API

Ollama exposes an OpenAI-compatible endpoint, so most LLM tooling can point straight at it. A quick generate call:

```bash
curl http://{TRUENASIP}:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "Why is the sky blue?",
  "stream": false
}'
```


## 2.4 Add a Web UI (Open WebUI)

Ollama has no frontend on its own. Open WebUI gives you a clean ChatGPT-style interface and connects over the API. Add it to the same compose stack:

```yaml
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    ports:
      - "3000:8080"
    volumes:
      - /mnt/tank/configs/open-webui:/app/backend/data
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
    depends_on:
      - ollama
    restart: unless-stopped
```

Browse to `http://<host-ip>:3000`, create the first admin account, and your installed models appear in the model dropdown automatically.

# 3 · Find the Right Model with llmfit
 
**llmfit** is a terminal tool that detects your hardware (CPU, RAM, GPU/VRAM) and ranks which models will actually run well on it before you download a thing. It pairs naturally with Ollama: from its TUI you can press <kbd>d</kbd> on any model to pull it straight into Ollama over the API — no manual `ollama pull`, no guessing at sizes.
 

## 3.1 Install llmfit
 
The installer drops a self-contained binary into your user directory with no root required:
 
```bash
curl -fsSL https://llmfit.axjns.dev/install.sh | sh -s -- --local
```
 
This installs to `~/.local/bin/llmfit`. Launch the interactive TUI with:
 
```bash
llmfit
```
 
The status bar across the top shows your detected specs — CPU, available/total RAM, GPU (or `none`), and which runtimes it found (Ollama, llama.cpp, MLX, Docker, LM Studio).
 
> 
> The TrueNAS root filesystem is wiped on every system update, so anything in `~/.local/bin` won't survive an upgrade. For a persistent install, download the release binary from the [GitHub releases page](https://github.com/AlexsJones/llmfit/releases) onto your pool (e.g. `/mnt/tank/configs/llmfit/llmfit`), make it executable with `chmod +x`, and run it from there — or simply re-run the one-liner after each upgrade.
{.is-warning}
 
## 3.2 Connect llmfit to Ollama
 
llmfit talks to Ollama over the same REST API on port `11434`. As long as Ollama's port is published to the host (the `11434:11434` mapping from section 1), llmfit finds it automatically at `localhost:11434` — the status bar flips to **Ollama: ✓** with a count of installed models, and each model already pulled gets a green **✓** in the **Inst** column.
 
If Ollama runs on another machine or a non-default port, point llmfit at it explicitly:
 
```bash
OLLAMA_HOST="http://<host-ip>:11434" llmfit
```

 
## 3.3 Read the Table
 
Models are ranked by a composite **Score** (quality, speed, fit, and context combined) by default. But the column that decides whether a model is *usable* depends on your hardware:
 
| Column | What to watch |
|--------|---------------|
| **Score** | Overall ranking — quality-weighted, good first sort |
| **tok/s** | Estimated generation speed. On CPU this is the real usability gate |
| **Mode** | `GPU` / `MoE` / `CPU+GPU` / `CPU` — how the model will run |
| **Fit** | `Perfect` / `Good` / `Marginal` / `Too Tight` memory verdict |
| **Mem %** | How much of your available memory the model uses |

 
> 
> On a **CPU-only** box, every model shows `CPU` mode and caps at `Marginal` fit — that's by design, not an error. Don't chase the top Score; watch **tok/s** instead. As a feel: 15+ tok/s is comfortable, ~5 is readable, and 1–2 tok/s is painful for interactive use. Mixture-of-Experts models (e.g. `Qwen3-Coder-30B-A3B`) are the sweet spot — they deliver large-model quality at small-model speed because only a few billion parameters are active per token.
{.is-success}
 
Press <kbd>S</kbd> to open hardware simulation and type in a VRAM figure (e.g. a 12 GB RTX 3060) to instantly recompute the whole table against that GPU — the clearest way to preview what adding a card would unlock before you buy one.
 
## 3.4 Pull the Best Model with the `d` Key
 
1. Navigate to a model with the arrow keys or <kbd>j</kbd> / <kbd>k</kbd>. Use <kbd>/</kbd> to search or <kbd>s</kbd> to cycle the sort column.
2. With your pick highlighted, press <kbd>d</kbd>. llmfit sends a `POST /api/pull` to Ollama and shows an animated progress bar on the row.
3. The download runs *inside* the Ollama container and lands in your `/mnt/tank/configs/ollama` volume — llmfit is only the trigger.
4. When it completes, the row gains a green **✓** in the **Inst** column, and the model appears automatically in the Open WebUI dropdown (section 2.4), ready to chat.
Common keys in the model table:
 
| Key | Action |
|-----|--------|
| <kbd>↑</kbd> / <kbd>↓</kbd> or <kbd>j</kbd> / <kbd>k</kbd> | Navigate models |
| <kbd>/</kbd> | Search by name, provider, params, or use case |
| <kbd>s</kbd> | Cycle sort column (Score, tok/s, Params, Mem%, Ctx…) |
| <kbd>f</kbd> | Cycle fit filter (All, Runnable, Perfect, Good, Marginal) |
| <kbd>d</kbd> | Download highlighted model into Ollama |
| <kbd>r</kbd> | Refresh installed models from Ollama |
| <kbd>S</kbd> | Hardware simulation (override RAM/VRAM/CPU) |
| <kbd>Enter</kbd> | Toggle the detail view for the selected model |
| <kbd>q</kbd> | Quit |

 




