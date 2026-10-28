---
title: Meridian
description: A guide to deploying Meridian
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-04-22T21:18:58.613Z
---

# What is Meridian?

**Meridian** is a local proxy that exposes the Claude Agent SDK (formerly the Claude Code SDK) as a standard Anthropic API endpoint, so third-party coding tools can talk to Claude through a Claude Max subscription instead of a metered API key. It runs on `127.0.0.1:3456` by default and speaks both the Anthropic Messages API and the OpenAI chat-completions API, which means tools like OpenCode, Crush, Droid, Cline, Aider, Pi, ForgeCode and Open WebUI can point at it with nothing more than a `base_url` change.

Newer releases also include a web UI at `/` (with **Providers** and **Settings** pages) and a preview **Antigravity** backend that routes requests through Google's official `agy` CLI instead of Claude. It is selected with `MERIDIAN_BACKEND=antigravity`; see the [Antigravity docs](https://github.com/rynfar/meridian/blob/HEAD/docs/antigravity.md).

Everything flows through the SDK's documented `query()` function — no OAuth interception, no patched binaries. Session management, streaming, prompt caching and rate limiting stay under Anthropic's control; Meridian only translates the SDK's output into the API shape that other tools expect. TypeScript, MIT licensed, distributed on npm as `@rynfar/meridian`.

> **Terms of service caveat.** Claude Max is licensed as an individual subscription for Anthropic's own clients. Using it to drive third-party tools is a grey area, and the project's own docs describe sharing one subscription across machines or teams — that part is squarely against Anthropic's terms and is the fastest way to get an account flagged. Keep this to your own account on your own machine.
{.is-warning}

<div class="glance">
  <div><span>Port</span><b><code>3456</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
  <div class="glance-links">
    <a href="https://github.com/rynfar/meridian"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>



# <img src="/docker.png" class="tab-icon"> 1 · Deploy Meridian

```yaml
services:
  meridian:
    image: ghcr.io/rynfar/meridian:latest
    container_name: meridian
    user: "568:568"
    environment:
      - HOME=/home/claude
      - MERIDIAN_TELEMETRY_PERSIST=1
      - MERIDIAN_HOST_ID=meridian-local # stable, unique per container
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
| `MERIDIAN_SONNET_MODEL` | `sonnet` | Sonnet 4.x only: `sonnet` (200k) or `sonnet[1m]` (1M, bills as Extra Usage). No effect on Sonnet 5+ |
| `MERIDIAN_OPUS_MODEL` | `opus[1m]` | Set to `opus` to stay on the 200k Opus variant |
| `MERIDIAN_1M_CONTEXT_SUPPORT` | unset | `0` stops Meridian from requesting any optional `[1m]` tier |
| `MERIDIAN_HOST_ID` | derived | Stable, unique container identity for session locks. Set it in containers |
| `MERIDIAN_BACKEND` | Claude | `antigravity` selects the preview Antigravity backend |
| `MERIDIAN_DEFAULT_AGENT` | `opencode` | Adapter for unrecognised clients. Requires restart. |
| `MERIDIAN_DEFER_TOOL_THRESHOLD` | `15` | Tool count before non-core tools defer via ToolSearch. `0` disables. |
| `MERIDIAN_TELEMETRY_PERSIST` | unset | Enable SQLite telemetry that survives restarts |
| `MERIDIAN_TELEMETRY_RETENTION_DAYS` | `7` | Telemetry retention window |
| `MERIDIAN_PROFILES` | unset | JSON array of profile configs, overrides disk discovery |
{.dense}

> Sonnet 5 and later (the default) run with a native 1M context window on every plan, with no Extra Usage. The `MERIDIAN_SONNET_MODEL` 200k default only matters if you pin a Sonnet 4.x model, where `sonnet[1m]` is always billed as Extra Usage on Max plans. Opus 1M is included with Max at no extra cost.
{.is-info}

## 2.3 Passthrough vs internal mode

The question is who actually executes the tools.

- **Passthrough** (default for coding agents) — Claude generates the tool call, Meridian hands it back to the client, and the client runs it with its own sandboxing and file tracking. This is what OpenCode, Crush, Cline and Claude Code all want.
- **Internal** — the SDK executes tools directly on the host and runs its own agent loop. This is for pure chat frontends like Open WebUI that have no tools of their own.

The adapter picks the right mode automatically. Override with `MERIDIAN_PASSTHROUGH=1` or `=0`.

> **Known limitation.** In passthrough mode the SDK is capped at one turn per request by default, so the client runs the tool calls and re-sends after each round. Set `MERIDIAN_PASSTHROUGH_MAX_TURNS` to raise the budget (with `MERIDIAN_PASSTHROUGH_EARLY_STOP=1`, the default, it still stops once a tool batch is handed to the client). See the [configuration docs](https://github.com/rynfar/meridian/blob/HEAD/docs/configuration.md).
{.is-warning}


