---
title: Whoogle
description: A guide to deploying Whoogle
published: true
date: 2026-01-15T15:32:30.903Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:09:32.431Z
---

# ![](/whoogle.png){class="tab-icon"} What is Whoogle?

Get Google search results, but without any ads, JavaScript, AMP links, cookies, or IP address tracking. 
# 1 · Deploy Whoogle
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  whoogle-search:
    image: ${WHOOGLE_IMAGE:-benbusby/whoogle-search}
    container_name: whoogle-search
    restart: unless-stopped
    pids_limit: 50
    mem_limit: 256mb
    memswap_limit: 256mb
    user: whoogle
    security_opt:
      - no-new-privileges
    cap_drop:
      - ALL
    tmpfs:
      - /config/:size=10M,uid=927,gid=927,mode=1700
      - /var/lib/tor/:size=15M,uid=927,gid=927,mode=1700
      - /run/tor/:size=1M,uid=927,gid=927,mode=1700
    ports:
      - 5000:5000
```


## <img src="/truenas.png" class="tab-icon"> TrueNAS
1. No changes needed, just deploy!

# 2 · Add Custom CSS
To make Whoogle look better, try some of the [user submitted CSS from GitHub!](https://github.com/benbusby/whoogle-search/wiki/User-Contributed-CSS-Themes)

(I like [Google Dark](https://github.com/benbusby/whoogle-search/wiki/User-Contributed-CSS-Themes#google-dark-by-treyg----screenshots))