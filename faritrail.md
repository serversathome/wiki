---
title: Fairtrail
description: A guide to deploying fairtrail
published: true
date: 2026-03-13T17:52:31.676Z
tags: 
editor: markdown
dateCreated: 2026-03-13T17:52:31.676Z
---

# What is Fairtrail?

**Fairtrail** is a self-hosted flight price tracking application that monitors airfare and alerts you when prices change. It runs entirely on your own hardware — no account required and no data leaves your machine. Fairtrail uses a built-in Chromium browser to scrape flight data, stores it in PostgreSQL, and caches with Redis. It also integrates with LLM providers (Anthropic, OpenAI, or Google AI) for AI-powered price analysis.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Fairtrail

```yaml
services:
  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_DB: fairtrail
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - /mnt/tank/configs/fairtrail/pgdata:/var/lib/postgresql/data
    ports:
      - "127.0.0.1:5433:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5

  redis:
    image: redis:7-alpine
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/fairtrail/redisdata:/data
    ports:
      - "127.0.0.1:6380:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5

  web:
    image: ghcr.io/affromero/fairtrail:latest
    restart: unless-stopped
    depends_on:
      db:
        condition: service_healthy
    ports:
      - "3003:3003"
    environment:
      DATABASE_URL: postgresql://postgres:postgres@db:5432/fairtrail
      REDIS_URL: redis://redis:6379
      CHROME_PATH: /usr/bin/chromium-browser
      NODE_ENV: production
      SELF_HOSTED: "true"
      CRON_SECRET: CHANGE_ME_TO_SOMETHING_RANDOM
      CLAUDE_CODE_ENABLED: "true"
    volumes:
      - /mnt/tank/configs/fairtrail/app-data:/app/data
      - ~/.claude:/home/node/.claude:ro
```

 
> Replace `CHANGE_ME_TO_SOMETHING_RANDOM` with a real secret. Generate one with:
> ```bash
> head -c 32 /dev/urandom | xxd -p | head -c 32
> ```
<!-- {blockquote:.is-warning} -->

# 2 · Configuration

## 2.1 LLM Provider
 
Fairtrail can integrate with an LLM for AI-powered price analysis. You have three options:
 
**Option A — Claude Code CLI (no API key needed)**
 
If you have Claude Code installed on your host, mount the credential directory into the container as a read-only volume and set `CLAUDE_CODE_ENABLED` to `true`. The compose file above is already configured for this. If you also have a `~/.claude.json` file, add this additional volume mount:
 
```yaml
      - ~/.claude.json:/home/node/.claude.json:ro
```
 
> If `~/.claude.json` does not exist on your host, **do not** add the volume mount. Docker will create it as an empty directory, which will cause errors.
<!-- {blockquote:.is-danger} -->
 
**Option B — OpenAI Codex CLI (no API key needed)**
 
If you have the Codex CLI installed on your host (`~/.codex` directory), remove the Claude-specific lines from the compose file and replace them with:
 
```yaml
    environment:
      # ...existing env vars...
      CODEX_ENABLED: "true"
    volumes:
      # ...existing volumes...
      - ~/.codex:/home/node/.codex:ro
```
 
**Option C — API key**
 
If you don't have Claude Code or Codex CLI installed, you can use an API key from any of the supported providers. Remove `CLAUDE_CODE_ENABLED` and the `~/.claude` volume mount, and add one of the following environment variables to the `web` service:
 
| Provider | Environment Variable | Where to get a key |
|---|---|---|
| Anthropic | `ANTHROPIC_API_KEY=sk-ant-your-key-here` | [console.anthropic.com](https://console.anthropic.com/) |
| OpenAI | `OPENAI_API_KEY=sk-your-key-here` | [platform.openai.com/api-keys](https://platform.openai.com/api-keys) |
| Google AI | `GOOGLE_AI_API_KEY=your-key-here` | [aistudio.google.com/apikey](https://aistudio.google.com/apikey) |
 
> All credential mounts are read-only (`:ro`). The container cannot modify your tokens, and nothing is copied or sent anywhere.
<!-- {blockquote:.is-success} -->
