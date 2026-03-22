---
title: Searxng
description: A guide to deploying Searxng
published: true
date: 2026-03-22T12:08:48.277Z
tags: 
editor: markdown
dateCreated: 2026-03-22T12:08:48.277Z
---

# <img src="/searxng.png" class="tab-icon"> What is SearXNG?

**SearXNG** is a free, open-source metasearch engine that aggregates results from over 240 search services — including Google, Bing, DuckDuckGo, Brave, and Wikipedia — without tracking users, storing cookies, or building profiles. It acts as a privacy-preserving proxy between you and search engines, stripping tracking data from requests and results.

SearXNG is a community-driven fork of the original SearX project, started in 2021, and is under very active development with 26,900+ GitHub stars.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy SearXNG
# {.tabset}
## Docker

```yaml
services:
  searxng:
    image: searxng/searxng:latest
    container_name: searxng
    environment:
      - SEARXNG_BASE_URL=http://localhost:5080/
      - SEARXNG_SECRET=${SEARXNG_SECRET}
    restart: unless-stopped
    ports:
      - "5080:8080"
    volumes:
      - /mnt/tank/configs/searxng:/etc/searxng:rw
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - SETGID
      - SETUID
      - DAC_OVERRIDE
    logging:
      driver: "json-file"
      options:
        max-size: "1m"
        max-file: "1"

  redis:
    image: redis:alpine
    container_name: searxng-redis
    command: redis-server --save "" --appendonly "no"
    restart: unless-stopped
    tmpfs:
      - /var/lib/redis
    cap_drop:
      - ALL
    cap_add:
      - SETGID
      - SETUID
      - DAC_OVERRIDE
```

1. Generate a secret key before deploying:
```bash
openssl rand -hex 32
```
2. Set the `SEARXNG_SECRET` environment variable with the generated key. You can either replace `${SEARXNG_SECRET}` directly or create a `.env` file in the same directory.
3. If you plan to access SearXNG from a domain name, update `SEARXNG_BASE_URL` to match (e.g., `https://search.yourdomain.com/`).

> 
> The `cap_drop: ALL` with selective `cap_add` follows the principle of least privilege, limiting what the container can do on the host system.
{.is-info}

> 
> If your pool is named something other than `tank`, change the left side of the volume path to match your pool name, e.g., `/mnt/yourpool/configs/searxng`.
{.is-warning}

## Docker + VPN (Gluetun)

This deployment routes all of SearXNG's outbound search traffic through a VPN using [Gluetun](https://github.com/qdm12/gluetun), so search engines see a shared VPN IP instead of your home IP. This is the **recommended setup** for single-user instances.

```yaml
services:
  gluetun:
    image: qmcgaw/gluetun:latest
    container_name: searxng-vpn
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
      - VPN_SERVICE_PROVIDER=airvpn
      - VPN_TYPE=wireguard
      - WIREGUARD_PRIVATE_KEY=${WIREGUARD_PRIVATE_KEY}
      - WIREGUARD_PRESHARED_KEY=${WIREGUARD_PRESHARED_KEY}
      - WIREGUARD_ADDRESSES=${WIREGUARD_ADDRESSES}
      - SERVER_COUNTRIES=Netherlands
    ports:
      - "5080:8080"
    restart: unless-stopped

  searxng:
    image: searxng/searxng:latest
    container_name: searxng
    network_mode: "service:gluetun"
    depends_on:
      - gluetun
    environment:
      - SEARXNG_BASE_URL=http://localhost:5080/
      - SEARXNG_SECRET=${SEARXNG_SECRET}
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/searxng:/etc/searxng:rw
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - SETGID
      - SETUID
      - DAC_OVERRIDE
    logging:
      driver: "json-file"
      options:
        max-size: "1m"
        max-file: "1"

  redis:
    image: redis:alpine
    container_name: searxng-redis
    network_mode: "service:gluetun"
    depends_on:
      - gluetun
    command: redis-server --save "" --appendonly "no"
    restart: unless-stopped
    tmpfs:
      - /var/lib/redis
    cap_drop:
      - ALL
    cap_add:
      - SETGID
      - SETUID
      - DAC_OVERRIDE
```

1. Generate a secret key before deploying:
```bash
openssl rand -hex 32
```
2. Set `SEARXNG_SECRET` in a `.env` file or replace it directly.
3. Replace the Gluetun VPN variables with your own provider credentials. The example above uses AirVPN with WireGuard. See the [Gluetun wiki](https://github.com/qdm12/gluetun-wiki) for your provider's specific variables.
4. The `ports` mapping is on the **Gluetun container**, not SearXNG — because SearXNG uses `network_mode: "service:gluetun"`, all its traffic flows through the VPN tunnel.

> 
> **How this works:** `network_mode: "service:gluetun"` forces SearXNG and Redis to share Gluetun's network stack. All outbound connections from SearXNG — every query to Google, Bing, Brave, etc. — go through the VPN tunnel. If the VPN drops, the built-in killswitch in Gluetun blocks all traffic, so no queries ever leak your real IP.
{.is-info}

> 
> If your pool is named something other than `tank`, change the left side of the volume path to match your pool name, e.g., `/mnt/yourpool/configs/searxng`.
{.is-warning}

> 
> Change `VPN_SERVICE_PROVIDER`, `VPN_TYPE`, and the credential variables to match your VPN provider. Gluetun supports 30+ providers including Mullvad, ProtonVPN, Surfshark, NordVPN, and more.
{.is-info}

## TrueNAS

1. Navigate to **Apps** in the TrueNAS UI
2. Click **Discover** and search for "**SearXNG**"
3. Click **Install**
4. Configure the following settings:
   - **Network > Web Port**: `5080` (or your preferred port)
   - **Storage > SearXNG Config Storage > Host Path**: `/mnt/tank/configs/searxng`
5. Click **Install**

> 
> SearXNG is available in the **Community** train. Make sure the Community train is enabled in your Apps settings.
{.is-info}

> 
> The TrueNAS app automatically includes Redis and generates a secret key for you.
{.is-success}

# 2 · Configuration

## 2.1 Initial Setup

Once SearXNG is running, access it at `http://YOUR-SERVER-IP:5080`. You'll see a clean search interface.

Click the **Preferences** gear icon to customize your instance:

| Setting | Recommendation | Why |
|---------|---------------|-----|
| Autocomplete | **Off** | Sends partial queries to engines as you type |
| Default theme | Simple (dark) | Clean, distraction-free |
| Results in new tabs | **On** | Keeps your search page open |
| SafeSearch | Your preference | Filters explicit content |
{.dense}

## 2.2 Search Engines

Under **Preferences > Engines**, you can enable or disable any of the 240+ search engines SearXNG supports. Engines are organized by category:

- **General**: Google, Bing, DuckDuckGo, Brave, Startpage, Qwant
- **Images**: Google Images, Bing Images, Flickr
- **Videos**: YouTube, PeerTube, Dailymotion
- **News**: Google News, Bing News, Yahoo News
- **Science**: Semantic Scholar, PubMed, arXiv
- **IT**: Stack Overflow, GitHub, npm, PyPI

> 
> Enabling too many engines simultaneously can slow down results and increase the chance of rate limiting. Start with 3-5 engines per category.
{.is-warning}

## 2.3 Settings File

For advanced configuration, edit `settings.yml` in your mounted config volume (`/mnt/tank/configs/searxng/settings.yml`).

Key settings include:

```yaml
search:
  safe_search: 0        # 0=off, 1=moderate, 2=strict
  default_lang: "en"
  autocomplete: false

server:
  secret_key: "your-generated-key"
  base_url: "http://localhost:5080/"

ui:
  default_theme: simple
  theme_args:
    simple_style: dark
```

> 
> After modifying `settings.yml`, restart the SearXNG container for changes to take effect.
{.is-info}

## 2.4 Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `SEARXNG_SECRET` | Cryptographic secret key for sessions | *required* |
| `SEARXNG_BASE_URL` | Public URL of your instance | `http://localhost:5080/` |
| `BIND_ADDRESS` | Address and port to bind inside container | `[::]:8080` |
| `SEARXNG_SETTINGS_PATH` | Path to settings file inside container | `/etc/searxng/settings.yml` |
{.dense}

> 
> Full environment variable documentation: [SearXNG Admin Docs](https://docs.searxng.org/admin/settings/index.html)
{.is-info}

# 3 · Privacy Guide

## 3.1 How SearXNG Protects You

SearXNG protects privacy in three ways:

1. **Strips private data from requests** — No cookies are sent to search engines. A random browser profile is generated for every request.
2. **Removes tracking from results** — Tracking scripts, analytics, and ad content are removed from results pages.
3. **Hides your IP** — Search engines see the IP of your SearXNG instance, not your personal device.

## 3.2 Threat Model Levels

| Level | Goal | Setup | What It Protects Against |
|-------|------|-------|--------------------------|
| **Basic** | Break ad profile tracking | SearXNG only | Google/Bing building an ad profile tied to your account or browser |
| **Enhanced** | IP correlation protection | SearXNG + VPN (e.g., Gluetun) | Search engines correlating queries from a single-user instance via IP |
| **Maximum** | Full anonymity | SearXNG + Tor | ISP and search engines seeing any connection to your queries |
{.dense}

> 
> **Important**: SearXNG does not hide your queries from search engines. Google still knows *someone* searched for something — SearXNG breaks the link between those queries and *you*.
{.is-warning}

## 3.3 Setting Up as Default Browser Search

You can add SearXNG as your default search engine in most browsers:

- **Firefox**: Go to your SearXNG instance, right-click the address bar, and select "Add SearXNG"
- **Chrome/Brave**: Settings > Search engine > Manage search engines > Add. URL: `http://YOUR-IP:5080/search?q=%s`
- **Safari**: Use an extension like "xSearch" to set a custom search URL

# <img src="/youtube.png" class="tab-icon"> 4 · Video

https://youtu.be/VIDEO_ID_HERE