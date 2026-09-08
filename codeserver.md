---
title: Code Server
description: A guide to deploying Code Server
published: true
date: 2026-01-15T15:28:33.497Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:50.157Z
---

# ![](/coder.png){class="tab-icon"} What is Code Server?

Run VS Code on any machine anywhere and access it in the browser.

<div class="glance">
  <div><span>Port</span><b><code>8443</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/linuxserver/docker-code-server"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# 1 · Deploy Code Server
# {.tabset}

## <img src="/truenas.png" class="tab-icon"> TrueNAS
1. Set an **Additional Environment Variable** 
a. Name = PASSWORD
b. Value = *choose a password*
1. Set the **Code Server Project Storage** to *Host Path* and select your pool 

## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  code-server:
    image: lscr.io/linuxserver/code-server:latest
    container_name: code-server
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
      - PASSWORD=admin
    volumes:
      - ./config:/config
      - /mnt/tank:/config/workspace
    ports:
      - 8443:8443
    restart: unless-stopped
```