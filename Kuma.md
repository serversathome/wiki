---
title: Uptime Kuma
description: A guide to installing Uptime Kuma in TrueNAS Scale as well as docker via compose
published: true
date: 2025-10-08T08:27:10.274Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:37:43.656Z
---

![](https://wiki.hydrology.cc/kumadash.jpg)

# ![](/uptime-kuma.png){class="tab-icon"} What is Uptime Kuma?

Uptime Kuma is a web monitor tool that supports various monitors such as HTTP, DNS, MySQL, Postgres, Docker, and more.

# 1 · Deploy Uptime Kuma
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: kuma
    volumes:
      - /mnt/tank/configs/kuma:/app/data
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - 3001:3001
    restart: unless-stopped
```

## <img src="/truenas.png" class="tab-icon"> TrueNAS

![screenshot_from_2025-02-22_11-39-20.png](/screenshot_from_2025-02-22_11-39-20.png)

- The **App Config Storage** *Type of Storage* should be set to **Host Path** as described in [Folder Structure](/Folder-Structure).

# 2 · Uptime Kuma Configuration

I'm not going to include a guide here on how to use Uptime Kuma. Its pretty straight forward and I think Christian did a great job here. 

[https://youtu.be/tIazVdhsSqQ?si=u4A4-4AUqToEWCEl&t=198](https://youtu.be/tIazVdhsSqQ?si=u4A4-4AUqToEWCEl&t=198)

# <img src="/youtube.png" class="tab-icon"> YouTube Walkthrough

[https://youtu.be/7VrU\_zNrzqo](https://youtu.be/7VrU_zNrzqo)