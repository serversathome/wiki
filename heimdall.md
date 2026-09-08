---
title: Heimdall
description: A guide to deploying Heimdall
published: true
date: 2026-01-15T15:29:29.139Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:05:07.229Z
---

# ![](/heimdall.png){class="tab-icon"} What is Heimdall?
As the name suggests Heimdall Application Dashboard is a dashboard for all your web applications. It doesn't need to be limited to applications though, you can add links to anything you like.

Heimdall is an elegant solution to organise all your web applications. It’s dedicated to this purpose so you won’t lose your links in a sea of bookmarks.

Why not use it as your browser start page? It even has the ability to include a search bar using either Google, Bing or DuckDuckGo.

<div class="glance">
  <div><span>Port</span><b><code>1080</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/linuxserver/docker-heimdall"><i class="mdi mdi-github"></i>Project</a>
    <a href="https://youtu.be/-pm-F9dzYn0"><i class="mdi mdi-youtube"></i>Video walkthrough</a>
  </div>
</div>


# 1 · Deploy Heimdall
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  heimdall:
    image: lscr.io/linuxserver/heimdall:latest
    container_name: heimdall
    restart: unless-stopped
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
      - ALLOW_INTERNAL_REQUESTS=false #optional
    volumes:
      - /mnt/tank/configs/heimdall:/config
    ports:
      - 1080:80
      - 10443:443
```
By default, Heimdall blocks lookup requests to private or reserved IP addresses, if your instance is not exposed to the internet, or is behind some level of authentication, you can set `ALLOW_INTERNAL_REQUESTS` to true to allow requests to private IP addresses.

## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Set the **Heimdall Data Storage** to *Host Path* and select a dataset to store the config files

# <img src="/youtube.png" class="tab-icon"> 2 · Video
https://youtu.be/-pm-F9dzYn0
