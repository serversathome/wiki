---
title: Dozzle
description: A guide to deploying Dozzle on TrueNAS Scale and via Docker Compose
published: true
date: 2026-01-15T15:29:04.827Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:04:29.434Z
---

# ![](/dozzle.png){class="tab-icon"} What is Dozzle?

Dozzle is a small lightweight application with a web based interface to monitor Docker logs. It doesn’t store any log files. It is for live monitoring of your container logs only.

<div class="glance">
  <div><span>Port</span><b><code>8888</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://youtu.be/LNDkGBOfv6Y"><i class="mdi mdi-youtube"></i>Video walkthrough</a>
  </div>
</div>

# 1 · Deploy Dozzle
# {.tabset}

## <img src="/truenas.png" class="tab-icon"> TrueNAS
Since dozzle does not require any volumes, unless you want to change the port, leave all setting default

## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
 dozzle:
   container_name: dozzle
   image: amir20/dozzle:latest
   ports:
     - '8888:8080'
   restart: unless-stopped
   volumes:
     - /var/run/docker.sock:/var/run/docker.sock
```

# <img src="/youtube.png" class="tab-icon"> 2 · Video

[https://youtu.be/LNDkGBOfv6Y](https://youtu.be/LNDkGBOfv6Y)