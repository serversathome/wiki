---
title: Meridian
description: A guide to deploying Meridian
published: true
date: 2026-09-05T10:53:44.190Z
tags: 
editor: markdown
dateCreated: 2026-04-22T21:18:58.613Z
---

# What is Meridian?

**Meridian** is a local proxy that exposes the Claude Code SDK as a standard Anthropic API endpoint, so third-party coding tools can talk to Claude through a Claude Max subscription instead of a metered API key. It runs on `127.0.0.1:3456` by default and speaks both the Anthropic Messages API and the OpenAI chat-completions API, which means tools like OpenCode, Crush, Droid, Cline, Aider, Pi, ForgeCode and Open WebUI can point at it with nothing more than a `base_url` change.

Everything flows through the SDK's documented `query()` function — no OAuth interception, no patched binaries. Session management, streaming, prompt caching and rate limiting stay under Anthropic's control; Meridian only translates the SDK's output into the API shape that other tools expect. TypeScript, MIT licensed, distributed on npm as `@rynfar/meridian`.

> **Terms of service caveat.** Claude Max is licensed as an individual subscription for Anthropic's own clients. Using it to drive third-party tools is a grey area, and the project's own docs describe sharing one subscription across machines or teams — that part is squarely against Anthropic's terms and is the fastest way to get an account flagged. Keep this to your own account on your own machine.
{.is-warning}



# <img src="/docker.png" class="tab-icon"> 1 · Deploy Meridian

```yaml
services:
  meridian:
    image: ghcr.io/rynfar/meridian:1.67
    container_name: meridian
    user: "568:568"
    environment:
      - HOME=/home/claude
      - MERIDIAN_API_KEY=change-me-to-a-long-random-string
      - MERIDIAN_TELEMETRY_PERSIST=1
    restart: unless-stopped
    ports:
      - "3456:3456"
    volumes:
      - /mnt/tank/configs/meridian:/home/claude
```


## 1.1 Seeding credentials

On the host, run this command to shell into the container:

```bash

docker exec -it meridian claude login
```
Then follow the standard steps to login to Claude Code.


# 2 · Configuration

## 2.1 Pointing tools at Meridian
To use Meridian, point the tools at `http://{IP}:3456` and enter any API key since Meridian will accept anything with no explicit key set in the environment variables. 


## 2.2 Environment variables

| Variable | Default | Description |
|----------|---------|-------------|
| `MERIDIAN_API_KEY` | unset | Shared secret. When set, all API and admin routes require `x-api-key` or `Authorization: Bearer`. `/` and `/health` stay open. |
| `MERIDIAN_PORT` | `3456` | Port to listen on |
| `MERIDIAN_HOST` | `127.0.0.1` | Bind address |
| `MERIDIAN_PASSTHROUGH` | unset | `1` forces tool calls back to the client, `0` forces the SDK to execute them |
| `MERIDIAN_MAX_CONCURRENT` | `10` | Maximum concurrent SDK sessions |
| `MERIDIAN_MAX_SESSIONS` | `1000` | In-memory LRU session cache size |
| `MERIDIAN_MAX_STORED_SESSIONS` | `10000` | File-based session store capacity |
| `MERIDIAN_WORKDIR` | `cwd()` | Default working directory for the SDK |
| `MERIDIAN_SONNET_MODEL` | `sonnet` | `sonnet` (200k) or `sonnet[1m]` (1M, bills as Extra Usage) |
| `MERIDIAN_DEFAULT_AGENT` | `opencode` | Adapter for unrecognised clients. Requires restart. |
| `MERIDIAN_DEFER_TOOL_THRESHOLD` | `15` | Tool count before non-core tools defer via ToolSearch. `0` disables. |
| `MERIDIAN_TELEMETRY_PERSIST` | unset | Enable SQLite telemetry that survives restarts |
| `MERIDIAN_TELEMETRY_RETENTION_DAYS` | `7` | Telemetry retention window |
| `MERIDIAN_PROFILES` | unset | JSON array of profile configs, overrides disk discovery |
{.dense}

> Sonnet defaults to 200k context on purpose — Sonnet 1M is always billed as Extra Usage on Max plans, even when regular usage is not exhausted. Opus 1M is included with Max at no extra cost.
{.is-info}

## 2.3 Passthrough vs internal mode

The question is who actually executes the tools.

- **Passthrough** (default for coding agents) — Claude generates the tool call, Meridian hands it back to the client, and the client runs it with its own sandboxing and file tracking. This is what OpenCode, Crush, Cline and Claude Code all want.
- **Internal** — the SDK executes tools directly on the host and runs its own agent loop. This is for pure chat frontends like Open WebUI that have no tools of their own.

The adapter picks the right mode automatically. Override with `MERIDIAN_PASSTHROUGH=1` or `=0`.

> **Known limitation.** In passthrough mode the SDK runs with `maxTurns=2`, so you get a single tool round-trip per request. Multi-step agentic loops need the client to re-send after each round.
{.is-warning}


