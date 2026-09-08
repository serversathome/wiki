---
title: OptiStack
description: A guide to deploying OptiStack
published: true
date: 2026-02-03T17:41:26.170Z
tags: 
editor: markdown
dateCreated: 2026-02-03T17:40:33.481Z
---

# <img src="/optistack.png" class="tab-icon"> What is OptiStack?

A supplement and medication manager for biohackers and health optimizers. Track dosages, check for interactions with optional AI analysis, manage inventory, and generate doctor-ready PDF reports.

<div class="glance">
  <div><span>Port</span><b><code>3000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy OptiStack

```yaml
services:
  optistack:
    image: ghcr.io/tylermiranda/optistack:latest
    container_name: optistack
    environment:
      - PORT=3000
      - JWT_SECRET=generate-with-openssl-rand-base64-48
      - SESSION_SECRET=generate-with-openssl-rand-base64-48
      - ADMIN_USERNAME=admin
      - ADMIN_PASSWORD=changeme
      - FRONTEND_URL=http://localhost:3000
      - AI_PROVIDER=openrouter
      - OPENROUTER_API_KEY=
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - /mnt/tank/configs/optistack:/app/data
```


> Generate secrets with `openssl rand -base64 48` — if not set, random secrets are generated at startup which invalidates sessions on restart.
{.is-warning}

> Change the `FRONTEND_URL` to your IP address
{.is-info}


# 2 · First Login

1. Access the web UI at `http://your-server:3000`
2. Login with your configured `ADMIN_USERNAME` and `ADMIN_PASSWORD`

# 3 · AI Configuration (Optional)

OptiStack supports AI-powered interaction checking via OpenRouter (cloud) or Ollama (local).

**Cloud AI:** Add your OpenRouter API key to `OPENROUTER_API_KEY` (get one at https://openrouter.ai/keys)

**Local AI (Ollama):** Replace the AI environment variables with:
```yaml
- AI_PROVIDER=ollama
- OLLAMA_URL=http://host.docker.internal:11434
- OLLAMA_MODEL=llama3.1:8b
```

> If no AI provider is configured, AI features are hidden from the UI
{.is-info}


# <img src="/youtube.png" class="tab-icon"> 4 · Video

*No video yet — check back soon!*