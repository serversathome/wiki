---
title: Ollama
description: A guide to deploying Ollama
published: true
date: 2026-06-15T14:57:57.280Z
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



