---
title: Meridian
description: A guide to deploying Meridian
published: true
date: 2026-04-22T21:18:58.613Z
tags: 
editor: markdown
dateCreated: 2026-04-22T21:18:58.613Z
---

# What is Meridian?

**Meridian** is a local proxy that exposes your Claude Max or Pro subscription as a standard Anthropic-compatible (and OpenAI-compatible) API. It runs the Claude Code SDK under the hood and translates its output into the API formats that third-party tools expect — so you can point OpenCode, Cline, Aider, Crush, Droid, Open WebUI, Continue, and anything else that accepts a custom `base_url` at Meridian, and have those requests count against your subscription instead of billing pay-per-token against an API key.

Unlike older proxies that extracted OAuth tokens or patched binaries, Meridian only calls documented SDK functions. Anthropic retains control of authentication, rate limiting, prompt caching, and context management. It's a presentation layer, not a jailbreak.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Meridian

Meridian does not publish a prebuilt image, so you build it once from source. Authentication requires a browser, which containers don't have — so there's a one-time seeding step with a throwaway container before the stack will stay up.

## 1.1 Build the image

SSH into your host and clone the repo somewhere outside your Dockge stack directory:

```bash
cd /mnt/tank/stacks
git clone https://github.com/rynfar/meridian.git
cd meridian
docker build -t meridian:latest .
```

## 1.2 Create the stack directory and seed credentials

Create a Dockge stack directory and an empty `claude-auth` bind mount next to it. The container runs as UID 1000, so the directory must be owned by that UID:

```bash
mkdir -p /mnt/tank/stacks/meridian/claude-auth
cd /mnt/tank/stacks/meridian
sudo chown -R 1000:1000 claude-auth
```

Launch a throwaway container with that directory mounted and run `claude login` inside it:

```bash
docker run --rm -it \
  --user claude \
  --entrypoint /bin/sh \
  -v "$(pwd)/claude-auth:/home/claude/.claude" \
  meridian:latest
```

At the container shell, authenticate:

```bash
claude 
```

Follow the OAuth URL in a browser, paste the returned code back into the container shell, then exit. You should now see `.credentials.json` (mode 0600) inside `./claude-auth/` on the host.

> 
> Whichever Claude account you authenticate here is the account every connected client will draw quota from. If you're signed into multiple accounts in your browser, sign out of any you don't want used before running the OAuth flow.
{.is-warning}

## 1.3 Deploy the stack

Create the stack in Dockge with the following `compose.yaml`:

```yaml
services:
  meridian:
    image: meridian:latest
    container_name: meridian
    restart: unless-stopped
    environment:
      - MERIDIAN_HOST=0.0.0.0
      - MERIDIAN_PORT=3456
      # - MERIDIAN_API_KEY=change-me-to-a-long-random-string   # see §2.5 before deciding
      # - MERIDIAN_MAX_CONCURRENT=10                           # bump if many agents share this proxy
    ports:
      - "3456:3456"
    volumes:
      - ./claude-auth:/home/claude/.claude
```


> `MERIDIAN_HOST` is the address the proxy binds to **inside the container** — keep it `0.0.0.0`. To scope the service to a specific host interface, put the IP in the `ports:` mapping as `HOST_IP:HOST_PORT:CONTAINER_PORT` instead. See the troubleshooting section — getting this backwards causes a silent crash loop.
{.is-danger}

# 2 · Configuration

## 2.1 Pointing clients at Meridian

Most Anthropic-compatible tools just need two environment variables. Meridian does not validate API keys against Anthropic, so unless you've enabled `MERIDIAN_API_KEY` (see §2.5) any non-empty value works:

```bash
export ANTHROPIC_API_KEY=dummy
export ANTHROPIC_BASE_URL=http://<host>:3456
```

If you have enabled `MERIDIAN_API_KEY`, set `ANTHROPIC_API_KEY` to that shared secret on every client instead. OpenAI-compatible tools (Open WebUI, Continue) point at `http://<host>:3456` as the OpenAI base URL, same rules for the key. Per-client setup for OpenCode, Crush, Droid, Cline, Aider, ForgeCode, and Pi is documented in the [Meridian README](https://github.com/rynfar/meridian).

## 2.2 Updating Meridian

Since there's no registry image, updates are manual:

```bash
cd /mnt/tank/stacks/meridian
git pull
docker build -t meridian:latest .
```

Then restart the stack in Dockge. Your `claude-auth` bind mount persists across rebuilds.

## 2.3 Sharing one proxy across multiple clients

Meridian fingerprints conversations per-request, so multiple clients can share the same proxy without stepping on each other's sessions. A few things to keep in mind:

- All clients draw from the same Max/Pro quota. Three active agents is roughly three times the burn rate.
- Default concurrency is 10 SDK sessions; bump `MERIDIAN_MAX_CONCURRENT` if you routinely run more.
- If you publish the port to a LAN interface, tunnel, or VPN, read §2.5 on the API key and network exposure before deploying.
- Multiple Meridian containers pointing at the same `claude-auth` will race on OAuth refresh and corrupt the token state. One proxy, many clients.

## 2.4 Multi-profile setup

Meridian supports multiple Claude accounts via separate profile directories, selected at runtime with the `x-meridian-profile` header. Useful if you want to segregate personal work from a work/team account. See the [Multi-Profile section](https://github.com/rynfar/meridian#multi-profile-support) of the README.

## 2.5 API key and network exposure

`MERIDIAN_API_KEY` is unset in the default compose above. Whether to enable it depends on where Meridian is reachable from. The setting isn't about protecting sensitive data — Meridian stores none — it's about protecting your Claude Max/Pro quota from being burned by something you didn't authorise.

