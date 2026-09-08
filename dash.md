---
title: Dash.
description: A guide to deploying Dash.
published: true
date: 2026-01-26T18:47:13.398Z
tags: 
editor: markdown
dateCreated: 2026-01-26T14:16:20.520Z
---

# <img src="/dashdot.png" class="tab-icon"> What is Dash.?

A simple, modern server dashboard, primarily used by smaller private servers.

<div class="glance">
  <div><span>Port</span><b><code>3001</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Hardware accel</span><b>Nvidia</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
  <div class="glance-links">
    <a href="https://getdashdot.com/docs/configuration/basic"><i class="mdi mdi-book-open-variant"></i>Docs</a>
  </div>
</div>

# 1 · Deploy Dash.
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Without GPU
```yaml
services:
  dash:
    image: mauricenino/dashdot:latest
    restart: unless-stopped
    container_name: dash
    privileged: true
    ports:
      - '3001:3001'
    volumes:
      - /mnt/tank/configs/dashdot:/mnt/host:ro
    environment:
      DASHDOT_WIDGET_LIST: 'os,cpu,storage,ram,network'
```

## <img src="/docker.png" class="tab-icon"> With GPU
```yaml
services:
  dash:
    image: mauricenino/dashdot:nvidia
    restart: unless-stopped
    privileged: true
    deploy:
      resources:
        reservations:
          devices:
            - capabilities:
                - gpu
     ports:
      - '3001:3001'
    volumes:
      - /mnt/tank/configs/dashdot:/mnt/host:ro
    environment:
      DASHDOT_WIDGET_LIST: 'os,cpu,storage,ram,network,gpu'
```

# 2 · Environment Variables
See list of all environment variables [here](https://getdashdot.com/docs/configuration/basic).