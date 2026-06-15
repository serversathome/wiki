---
title: OpenWebUI
description: A guide to deploying OpenWebUI
published: true
date: 2026-06-15T15:33:22.377Z
tags: 
editor: markdown
dateCreated: 2026-06-15T15:23:06.042Z
---

# <img src="/open-webui.png" class="tab-icon"> What is Open WebUI?

**Open WebUI** is an extensible, self-hosted AI chat interface that turns local model runners (like Ollama) and any OpenAI-compatible API into a polished, multi-user web app. It runs entirely offline if you want it to, and out of the box it already covers most of what you'd expect from ChatGPT, Claude, or PewDiePie's Odysseus — web search, document chat (RAG), image generation, code execution, voice, and vision — all behind a clean UI you actually own.

The difference from the commercial tools is that everything here is a toggle or a plugin you control. The core ships with the "assistant" features built in; a Python plugin system (Tools, Functions, Pipelines) plus an MCP bridge lets you add anything the core doesn't have. This page covers the deploy, the built-in parity features, and the community extras worth installing.


# 1 · Deploy Open WebUI
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

This compose assumes you already have a model backend running (e.g. Ollama elsewhere on the LAN or an OpenAI-compatible endpoint). The container listens on **8080** internally; we map it to **3000** on the host.

```yaml
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: unless-stopped
    ports:
      - "3000:8080"
    environment:
      # Point at your model backend(s)
      - OLLAMA_BASE_URL=http://ollama:11434
      # Or use any OpenAI-compatible API (OpenRouter, your Meridian proxy, etc.)
      # - OPENAI_API_BASE_URL=http://10.0.0.20:4000/v1
      # - OPENAI_API_KEY=sk-xxxxxxxx
      # REQUIRED for stable sessions — generate with: openssl rand -hex 32
      - WEBUI_SECRET_KEY=change-me-to-a-long-random-string
      - WEBUI_AUTH=true
      # New signups land as "pending" until an admin approves them
      - DEFAULT_USER_ROLE=pending
    volumes:
      - /mnt/tank/configs/openwebui:/app/backend/data

```

1. Drop this into a new stack in **Dockge** and deploy.
2. Browse to `http://<host-ip>:3000` — **the first account you create becomes the admin.**
3. Approve any later signups under **Admin Panel → Users** (because of `DEFAULT_USER_ROLE=pending`).

> 
> Without a persistent `WEBUI_SECRET_KEY`, every container recreate logs out all users. Set it once and leave it. Generate one with `openssl rand -hex 32`.
{.is-warning}



## <img src="/truenas.png" class="tab-icon"> TrueNAS

Open WebUI is in the **Community** train (App Version v0.9.5 as of May 2026, requires TrueNAS 24.10.2.2 or newer). The TrueNAS app bundles a Redis sidecar for you.

1. Navigate to **Apps** in the TrueNAS UI and select the **Discover** screen.
2. Search for **Open WebUI** and click **Install**.
3. Configure the key settings:
   - **Web Port**: `30080` (or your choice)
   - **Ollama URL**: point at your existing Ollama app, e.g. `http://<ollama-app>:11434`
   - **Storage → Data**: set a **Host Path** of `/mnt/tank/configs/openwebui` (maps to `/app/backend/data`)
   - **Secret Key**: paste a long random string so sessions survive restarts
4. Click **Install** and wait for the app to report **Running**.
5. Open the **Web Portal** — the first account you create is the admin.

> 
> The Community train app runs the `open-webui` container as **root**. That's fine on a single-user homelab box, but worth noting if you're locking down a shared system. The bundled Redis sidecar runs as the non-root `apps` user (568).
{.is-info}

# 2 · First-Run & Core Setup

## 2.1 Connect your models

Go to **Admin Panel → Settings → Connections**. You can register as many backends as you like and they'll all appear in the model picker:

| Backend type | Where to set it |
|--------------|-----------------|
| Ollama (local models) | Ollama API field, e.g. `http://ollama:11434` |
| OpenAI / OpenRouter / LiteLLM / any OpenAI-compatible | OpenAI API field + key |
| Anthropic Claude | via a **Pipe Function** (see §5.2) |

Pull Ollama models directly from the UI under **Admin Panel → Settings → Models** — no shelling into the Ollama box required.

## 2.2 Lock down access

Open WebUI is multi-user from day one. Set per-model permissions, create user groups, and (on the enterprise side) wire up SSO/OIDC. For a homelab the important bits are: keep `DEFAULT_USER_ROLE=pending` so randoms behind your reverse proxy can't self-register into a working account, and only give **admin** to accounts that should be allowed to install Tools and Functions.


# 3 · Built-In Parity Features

These are the features that make Open WebUI feel like a commercial assistant. **None of them are plugins** — they ship in core and just need enabling/configuring under the Admin Panel. Start here before reaching for community extras.

## 3.1 Web search

Enable under **Admin Panel → Settings → Web Search**. Open WebUI supports a long list of providers — SearXNG (self-hosted, my pick), Brave, Google PSE, Tavily, DuckDuckGo, and others. Pair it with a self-hosted **SearXNG** instance and you get private, no-API-key web grounding that rivals ChatGPT Search.

## 3.2 Document chat / RAG (Knowledge)

Upload files inline with `#` in the chat, or build persistent **Knowledge** bases under the Workspace. Open WebUI chunks, embeds, and retrieves with hybrid search + reranking. Attach a Knowledge base to a custom model and you've got a Claude-Projects-style "this model always knows my docs" setup.

## 3.3 Image generation

Enable under **Admin Panel → Settings → Images**. Backends include **ComfyUI** and **AUTOMATIC1111** (local), plus OpenAI/Gemini image APIs. Once on, the model can generate images right in the chat the way ChatGPT does with DALL·E.

## 3.4 Code interpreter

A built-in **Code Interpreter** runs Python in a sandbox (Pyodide in-browser, or a Jupyter backend for heavier work). Combined with native function calling, the model can write, run, and iterate on code — the Odysseus/ChatGPT "analyse this CSV" experience.

## 3.5 Voice, vision & the rest

- **Voice**: STT + TTS with a hands-free **Call** mode (Whisper locally, or external providers).
- **Vision**: multimodal models can read images you paste in.
- **Memory**: opt-in long-term memory the assistant carries across chats.
- **Artifacts**: renders HTML/SVG/code output in a live side panel.
- **Channels & Notes**: a Discord-style chat space and a built-in notes app, both wired into the model.


# 4 · The Plugin System

When core doesn't cover it, the Python plugin system does. There are three layers, from easiest to most advanced.

## 4.1 Tools

**Tools** are Python functions the model calls *during* a chat — fetch weather, query a database, hit an API, control a device. The model decides when to call them via **function calling**. Install them in one click from the community hub, or write your own in the Workspace.

> 
> Open WebUI now uses **Native (Agentic) function calling** as the only supported mode. Enable it per model under **Model Settings → Advanced Params → Function Calling → Native**. The legacy "Default" prompt-injection mode still exists for backwards compatibility but is no longer maintained — don't build on it.
{.is-warning}

## 4.2 Functions

**Functions** extend Open WebUI *itself*, not just a single chat. Three subtypes:

| Type | What it does | Example |
|------|--------------|---------|
| **Pipe** | Creates a brand-new "model" in the picker | Add Anthropic Claude, a custom RAG flow, or a multi-step agent |
| **Filter** | Pre/post-processes every message (inlet/outlet/stream) | Live translation, token-usage display, toxic-message filtering, rate limiting |
| **Action** | Adds a custom button under a message | "Summarize this", "Send to webhook", "Re-run with different model" |

A **Pipe** function is how you get Claude or Gemini to show up as a selectable model. A **Filter** is how you bolt cross-cutting behavior onto everything.

## 4.3 Pipelines

**Pipelines** move heavy or advanced logic onto a *separate* server that Open WebUI talks to as if it were an OpenAI endpoint. You point the OpenAI URL at the Pipelines instance and it can do request routing, offloaded RAG, LangChain/LlamaIndex agents, usage monitoring with Langfuse, etc. The docs are explicit that **most people don't need Pipelines** — Functions cover the vast majority of cases now. Reach for it only when you're offloading real compute or running an external agent framework.

## 4.4 MCP (Model Context Protocol)

Open WebUI connects to **MCP servers**, so the same tool servers you'd use with Claude Desktop work here too. The standard path is the **`mcpo`** proxy, which exposes an MCP server as an OpenAPI endpoint Open WebUI can consume as a Tool. Some community toolkits (below) add richer native MCP handling with connection pooling and resilience.


# 5 · Community Extras

This is where you close the last gap to ChatGPT/Claude/Odysseus. Everything below is community-maintained.

## 5.1 The Community Hub

Browse and one-click import prompts, models, Tools, and Functions at **[openwebui.com](https://openwebui.com)**. It's the equivalent of an extension store — filter by Tools vs Functions, read the source, import straight into your Workspace. This is the first place to look for any capability you're missing.

## 5.2 Add Claude (and other providers) as models

Install an **Anthropic Pipe (manifold) Function** from the hub and paste your API key. Claude models then appear in the picker right next to your local Ollama models, with streaming and vision. The same pattern exists for Gemini, Azure OpenAI, and OpenRouter — handy if you run a mixed local/cloud setup (or front everything through your Meridian proxy and just register it as one OpenAI-compatible connection instead).

## 5.3 Power toolkits

- **[Haervwe/open-webui-tools](https://github.com/Haervwe/open-webui-tools)** — a modular toolkit of 15+ Tools, pipes, and filters that turns the instance into an agentic workstation: a Web Search Agent (research + citations), Image Gen Agent, Knowledge Agent (RAG over your docs + memory), Code Interpreter Agent, and a Terminal Agent for shell tasks. It also adds extended MCP server support with connection deduplication. This single project gets you closest to the Odysseus "autonomous agent" feel.
- **Adaptive memory / enhanced memory** functions — more capable long-term memory than the built-in toggle, if you want a Claude-style persistent profile.
- **Auto-tagging & autocomplete** functions — quality-of-life polish that matches the commercial UX.

## 5.4 Supporting services to self-host alongside

These aren't Open WebUI plugins but they're what the plugins/features hook into:

| Service | Powers | Notes |
|---------|--------|-------|
| **SearXNG** | Private web search (§3.1) | No API keys, no tracking — the homelab default |
| **ComfyUI** | Image generation (§3.3) | Most flexible local image backend |
| **Apache Tika / Docling** | Better document parsing for RAG | Improves §3.2 on messy PDFs |
| **mcpo** | MCP → OpenAPI bridge (§4.4) | Connects any MCP server as a Tool |

