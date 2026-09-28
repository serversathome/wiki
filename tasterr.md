---
title: Tasterr
description: A guide to deploying Tasterr
published: true
date: 2026-09-28T18:06:40.122Z
tags: 
editor: markdown
dateCreated: 2026-09-28T18:06:40.122Z
---

# <img src="/tasterr.svg" class="tab-icon"> What is Tasterr?

**Tasterr** is a Netflix-style discovery front end for your media stack. It pairs the TMDB catalog with your existing Seerr instance, so everyone in the house can browse, search, see what is already available, and request titles, all while Tasterr learns a separate taste profile for each person. Sign in with Plex and it also learns from your watch history, adds a Continue Watching row, and can blend recommendations for whoever is on the couch.

<div class="glance">
  <div><span>Port</span><b><code>8000</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/ZacharyArthur/tasterr"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

> Tasterr does not replace Seerr, it sits in front of it. You need a running Seerr instance, a Seerr API key, and a TMDB API key before you deploy.
{.is-info}

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Tasterr

## 1.1 Gather Your Keys

1. **Seerr API key**: in Seerr, go to **Settings** > **General** and copy the API key
2. **TMDB API key**: create a free account at themoviedb.org, then go to **Settings** > **API** and copy the v3 API key
3. **Tasterr secret key**: generate one in the TrueNAS shell. This encrypts stored Plex tokens

```bash
openssl rand -base64 32
```

## 1.2 Compose File



```yaml
services:
  tasterr:
    image: ghcr.io/zacharyarthur/tasterr:latest
    container_name: tasterr
    user: "568:568"
    environment:
      - TMDB_API_KEY=CHANGEME
      - SEERR_INTERNAL_URL=http://your-server-ip:5055
      - SEERR_EXTERNAL_URL=http://your-server-ip:5055
      - SEERR_API_KEY=CHANGEME
      - TASTERR_SECRET_KEY=CHANGEME
    ports:
      - "8000:8000"
    volumes:
      - /mnt/tank/configs/tasterr:/data
    restart: unless-stopped
```

1. Fill in the three keys
2. `SEERR_INTERNAL_URL` is how the Tasterr container reaches Seerr. Your server's LAN IP and Seerr's port works without any shared Docker network
3. `SEERR_EXTERNAL_URL` is the address your household's browsers use for Seerr. If Seerr is behind a Cloudflare Tunnel, use that public URL here
4. Deploy the stack. The SQLite database is created and migrated automatically on first boot

# 2 · Configuration

## 2.1 Initial Setup

1. Browse to `http://your-server-ip:8000`
2. Sign in with an existing Seerr account, or with Plex if Seerr uses Plex sign-in
3. Sign in as a **Seerr administrator** and open **Settings** to choose your region, the streaming services you pay for, which discovery rails appear on the home page, and the theme and accent color

Each person should sign in with their own account. Taste profiles and requests are tracked per user.

## 2.2 Plex Features

Signing in with Plex turns on extra features with no Plex URL or token to configure: recommendations that learn from watch history, a Continue Watching rail, and a household blend that mixes recommendations for several people at once. Local Seerr accounts still get discovery, search, requests and taste learning, just without the watch history signals.

## 2.3 Behind a Reverse Proxy or Tunnel

When TLS terminates at a proxy, Tasterr needs to trust that proxy's forwarded headers to mark session cookies `Secure`. Add the proxy's IP (or the narrowest subnet you can) to the environment:

```yaml
      - TASTERR_FORWARDED_ALLOW_IPS=172.18.0.10
```

> Never set `TASTERR_FORWARDED_ALLOW_IPS` to `*`. Also configure your proxy to strip or redact query strings from its access logs, since search URLs contain what your household is searching for.
{.is-danger}

## 2.4 If Something Is Missing

- **No catalog at all**: the TMDB key is missing or wrong
- **Availability shows Unknown and the request button is disabled**: Tasterr cannot reach Seerr at `SEERR_INTERNAL_URL`, or the Seerr API key is wrong
- **Health check**: `curl http://your-server-ip:8000/api/v1/health`

