---
title: Flaresolverr
description: A guide to installing Flaresolverr on TrueNAS and Ubuntu Server LTS
published: true
date: 2026-01-22T19:16:20.715Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:02:27.125Z
---

# ![](/flaresolverr.png){class="tab-icon"} What is Flaresolverr?
FlareSolverr is a proxy server to bypass Cloudflare and DDoS-GUARD protection.

# 1 · Deploy Flaresolverr
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS

![](/screenshot_from_2025-02-07_13-01-56.png)

Change the **WebUI Port** to **8191** since that is what is used in all the default documentation.

> You do not need a host path volume for this container
{.is-info}


## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  flaresolverr:
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr
    environment:
      - LOG_LEVEL=info
      - LOG_HTML=false
      - CAPTCHA_SOLVER=none
      - TZ=America/New_York
    ports:
      - 8191:8191
    restart: unless-stopped
```

# <img src="/youtube.png" class="tab-icon"> 2 · Video

https://youtu.be/sUMT0PTll_M