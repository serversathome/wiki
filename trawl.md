---
title: Trawl
description: A guide to deploying Trawl
published: true
date: 2026-10-22T10:07:06.786Z
tags: 
editor: markdown
dateCreated: 2026-10-20T15:12:05.236Z
---

# What is Trawl?

**Trawl** is a self-hosted scraping engine that gets past browser challenges like Cloudflare, Turnstile and DDoS-Guard. It speaks the same `/v1` API as FlareSolverr, so Prowlarr, Jackett and the rest of the \*arr stack can use it with a single URL change. Unlike FlareSolverr, it tries a plain HTTP request first, reuses clearance it has already earned, and only opens a browser (Camoufox, a hardened Firefox) when a site actually demands it. That makes repeat requests dramatically faster and much lighter on CPU.

On top of the FlareSolverr drop-in, Trawl adds an MCP server for AI agents, isolated browser sessions, a challenge-aware forward proxy and a local dashboard that shows every request it handled.

<div class="glance">
  <div><span>Port</span><b><code>8191</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/germondai/trawl"><i class="mdi mdi-github"></i>Project</a>
    <a href="https://docs.trawl.germondai.com"><i class="mdi mdi-book-open-variant"></i>Docs</a>
    <a href="https://apps.truenas.com/catalog/trawl_community/"><i class="mdi mdi-server"></i>TrueNAS app</a>
  </div>
</div>

# 1 · Deploy Trawl
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

Create the config folders and hand them to the apps user first:

```yaml
services:
  trawl:
    image: ghcr.io/germondai/trawl:latest
    container_name: trawl
    user: "568:568"
    shm_size: 1gb
    environment:
      - SESSION_CACHE_DRIVER=memory
      - BROWSER_POOL_SIZE=1
      - METRICS_DASHBOARD_ENABLED=true
      - MCP_ENABLED=true
    ports:
      - 8191:8191
    volumes:
      - /mnt/tank/configs/trawl:/data/metrics
    healthcheck:
      test: ["CMD", "curl", "-sf", "http://localhost:8191/health"]
      interval: 30s
      start_period: 90s
    restart: unless-stopped
```

1. Deploy the stack and give it a minute or so for the browser to warm up.
2. Check that it is healthy by browsing to `http://YOUR-SERVER-IP:8191/health`.
3. Open the dashboard at `http://YOUR-SERVER-IP:8191/dashboard`.

> On an older CPU or NAS processor, swap the image tag to `ghcr.io/germondai/trawl:baseline`. The standard image targets newer instruction sets and may crash on start otherwise.
{.is-info}

> Trawl has no login on its API, dashboard or MCP endpoint. Keep port 8191 on your LAN and never forward it through your router or publish it through a reverse proxy without authentication in front of it.
{.is-warning}

## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Navigate to **Apps** in the TrueNAS UI and click **Discover Apps**
2. Search for **Trawl** (it is in the Community train) and click **Install**
3. Configure the following settings:
   - **Redis Password**: set any strong password. The TrueNAS app bundles Redis for the clearance cache
   - **Browser Pool Size**: `1` is plenty for a typical \*arr stack. Raise it only if you see requests queueing
   - **Shared Memory Size (MiB)**: `1024`
   - **WebUI Port**: change the default `30460` to `8191` so it matches what FlareSolverr clients expect
   - **Additional Environment Variables**: add `METRICS_DASHBOARD_ENABLED` = `true` and `MCP_ENABLED` = `true`
   - **Additional Storage**: add a **Host Path** with Mount Path `/data/metrics` and Host Path `/mnt/tank/configs/trawl/metrics` so dashboard history survives updates
4. Click **Install**

> The TrueNAS app runs the Trawl container as root and the bundled Redis as the apps user (568). Leave the defaults alone here.
{.is-info}

# 2 · Connect Prowlarr

Trawl answers on the same FlareSolverr API, so nothing changes on the indexer side.

1. In Prowlarr, go to **Settings** > **Indexers**
2. Click **+** and choose **FlareSolverr**
3. Fill in:
   - **Name**: `Trawl`
   - **Tags**: `flaresolverr` (or whatever tag your protected indexers already use)
   - **Host**: `http://YOUR-SERVER-IP:8191/` (or `http://trawl:8191/` if Prowlarr is on the same Docker network)
4. Click **Test**, then **Save**
5. Make sure every Cloudflare-protected indexer carries the same tag

> If you are replacing FlareSolverr, just edit the existing FlareSolverr proxy and change the **Host** to point at Trawl. Your tags and indexers stay exactly as they are.
{.is-success}

> The first search against each indexer still pays the full challenge time. After that, Trawl reuses the clearance and searches come back fast. Some indexers serve results from a different domain than their homepage, and each hostname gets its own first solve.
{.is-info}


# 3 · Configuration

## 3.1 Dashboard

With `METRICS_DASHBOARD_ENABLED=true`, browse to `/dashboard` to see every request Trawl handled, which tier won (HTTP, cached session, live solve or proxy), how long it took and why anything failed. History is stored in SQLite under `/data/metrics`, which is why that folder is mapped.

## 3.2 MCP for AI agents

With `MCP_ENABLED=true`, any MCP client that supports Streamable HTTP can connect to:

```
http://YOUR-SERVER-IP:8191/mcp
```

| Tool | What it returns |
|------|-----------------|
| `read` | Main page content as Markdown or text |
| `scrape` | Full HTML with timing and tier info |
| `extract` | JSON records picked out with CSS selectors |
| `screenshot` | Viewport, full page or single element |
| `inspect` | Console, network activity and redirects |


Trawl loads URLs you give it. It does not do web search on its own.

## 3.3 Useful settings

| Variable | Default | What it does |
|----------|---------|--------------|
| `BROWSER_POOL_SIZE` | `1` | Warm browsers ready for concurrent requests. Each one costs RAM |
| `SESSION_CACHE_DRIVER` | `redis` | Use `memory` for a single container with no Redis |
| `SCRAPE_MAX_TIMEOUT_MS` | `60000` | How long a request may run before giving up |
| `TIER1_TRANSPORT` | `bun` | Set to `wreq` to impersonate Firefox TLS on plain HTTP requests |
| `PROXY_URL` | unset | Upstream proxy for all browser traffic |
| `RESIDENTIAL_PROXY_URL` | unset | Fallback proxy used only when the direct path fails |
{.dense}

The full list lives in the [configuration docs](https://docs.trawl.germondai.com/getting-started/configuration).

