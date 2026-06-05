---
title: Odysseus
description: A guide to deploying Odysseus
published: true
date: 2026-06-05T20:38:01.780Z
tags: 
editor: markdown
dateCreated: 2026-06-05T20:38:01.780Z
---

# <img src="/odysseus.png" class="tab-icon"> What is Odysseus?

**Odysseus** is a free, self-hosted AI workspace created by PewDiePie. Think of it as the ChatGPT or Claude website, but running on hardware you own. On top of chat it bundles a research agent, a model-picker called Cookbook, blind model comparison, persistent memory, plus email, notes, tasks, and a calendar — all in one app.

You bring your own "brain": point it at a **local model** (via Ollama, vLLM, llama.cpp, etc.) for full privacy and no usage cost, **or** plug in an **API key** (Anthropic/Claude, OpenAI/ChatGPT, OpenRouter) for top-tier models at a per-use cost.



# <img src="/docker.png" class="tab-icon"> 1 · Deploy Odysseus

Open the TrueNAS shell (or SSH in) and run:

```bash

cd /mnt/tank/stacks
git clone https://github.com/pewdiepie-archdaemon/odysseus.git
cd odysseus
cp .env.example .env
```

Now edit the `.env` file and set the values you care about (see the table in section 2), then build and start the stack:

```bash
docker compose up -d --build
```

The first build takes a few minutes while it pulls images and compiles the app. When it's done, all four containers should report healthy.




> **Do NOT expose this to the public internet.** Odysseus has an agent mode that can run commands on your server, with no sandbox (the project says so in its own docs). Keep it on your home network only. For remote access, use a VPN or a tunnel (Tailscale, Cloudflare Tunnel) — never a plain port-forward.
{.is-danger}

# 2 · Configuration

## 2.1 Key `.env` settings

Edit these in the `.env` file **before** your first `docker compose up`:

| Setting | What it does | Recommended |
|---------|--------------|-------------|
| `APP_PORT` | Web UI port on the host | `7000` (change if taken) |
| `APP_BIND` | Which addresses can reach the UI. `127.0.0.1` = this server only; `0.0.0.0` = your whole LAN | `0.0.0.0` for home-network access |
| `AUTH_ENABLED` | Login requirement | **Always `true`** |
| `LOCALHOST_BYPASS` | Dev-only auth skip for loopback | **Keep `false`** |
| `ODYSSEUS_ADMIN_USER` | Admin username | `admin` (default) |
| `ODYSSEUS_ADMIN_PASSWORD` | Pre-set your admin password | Set this so you don't have to hunt the logs |


After changing `.env`, apply it with:

```bash
docker compose up -d
```

## 2.2 First login (the password hides in the logs)

On first start, Odysseus creates the admin account and prints a **temporary password to the logs** — it does not show it on screen. If you didn't set `ODYSSEUS_ADMIN_PASSWORD` yourself, grab it from the logs:

```bash
docker compose logs odysseus | grep -i password
```

In Dockge, you can also just open the stack and read the log output. Log in with your admin user and that password, then change it in **Settings**.

> 
> The temporary password is printed **once**. Setting `ODYSSEUS_ADMIN_PASSWORD` in `.env` before first boot avoids the whole scavenger hunt.
{.is-info}

## 2.3 Connect a model (give it a brain)

Odysseus doesn't think on its own — you connect a model in **Settings**. Pick one of the two paths:

**Local (private, free to run):** If you have Ollama running, add this as an endpoint in Settings:

```text
http://host.docker.internal:11434/v1
```

Make sure Ollama is listening beyond its own loopback so the container can reach it. With a local model, nothing you type ever leaves your machine.

**API key (top-tier models, costs per use):** In Settings, add your provider's API key. Anthropic (Claude) is fully supported, as are OpenAI and OpenRouter.


# 3 · Cookbook & Local Models (Optional)

Cookbook is Odysseus's built-in helper for running local models. It scans your hardware, scores compatibility, and recommends models that will actually fit, then can download and serve them for you — great if you're new to local AI. It uses **tmux** for background downloads and serving (already included in the Docker image).

## 3.1 GPU support

Cookbook and local models run far better with a GPU. Install the host GPU runtime first, then add one line to `.env`:

```bash
# NVIDIA
COMPOSE_FILE=docker-compose.yml:docker/gpu.nvidia.yml

# AMD
COMPOSE_FILE=docker-compose.yml:docker/gpu.amd.yml
```

Recreate the stack, then verify the GPU is visible inside the container:

```bash
docker compose exec odysseus nvidia-smi -L   # NVIDIA
docker compose exec odysseus rocm-smi        # AMD
```


# 4 · Optional Extras (PDF / Office) — License Note

You can build with extra document features (PDF viewer, Office extraction):

```bash
docker compose build --build-arg INSTALL_OPTIONAL=true
docker compose up -d
```


# 5 · Updating

To pull the latest code and rebuild:

```bash
cd /mnt/bigdeal/stacks/odysseus
git pull
docker compose up -d --build
```

Your data in `./data` is preserved across updates. This is a fast-moving, year-one project, so expect occasional breaking changes — check the GitHub README before major updates.

# <img src="/youtube.png" class="tab-icon"> 6 · Video
