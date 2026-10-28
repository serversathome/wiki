---
title: Uptime Kuma
description: A guide to installing Uptime Kuma in TrueNAS Scale as well as docker via compose
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:02:35.330Z
---

![](https://wiki.hydrology.cc/kumadash.jpg)

# ![](/uptime-kuma.png){class="tab-icon"} What is Uptime Kuma?

Uptime Kuma is a web monitor tool that supports various monitors such as HTTP, DNS, MySQL, Postgres, Docker, and more.

<div class="glance">
  <div><span>Port</span><b><code>3001</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://youtu.be/tIazVdhsSqQ?si=u4A4-4AUqToEWCEl&t=198"><i class="mdi mdi-youtube"></i>Video walkthrough</a>
  </div>
</div>

# 1 · Deploy Uptime Kuma
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:2
    container_name: kuma
    volumes:
      - /mnt/tank/configs/kuma:/app/data
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - 3001:3001
    restart: unless-stopped
```

> **Upgrading from 1.x?** The `latest` tag still points to the old 1.x line, so switching to `:2` is a major upgrade. On first start, 2.x runs a one-time database migration that can take minutes to hours. Stop the container and back up `/mnt/tank/configs/kuma` first, then let the migration finish without interrupting it (watch `docker logs -f kuma`). See the [v1 to v2 migration guide](https://github.com/louislam/uptime-kuma/wiki/Migration-From-v1-To-v2).
{.is-warning}

## <img src="/truenas.png" class="tab-icon"> TrueNAS

![screenshot_from_2025-02-22_11-39-20.png](/screenshot_from_2025-02-22_11-39-20.png)

- The **App Config Storage** *Type of Storage* should be set to **Host Path** as described in [Folder Structure](/Folder-Structure).

# 2 · Uptime Kuma Configuration

I'm not going to include a guide here on how to use Uptime Kuma. Its pretty straight forward and I think Christian did a great job here. 

[https://youtu.be/tIazVdhsSqQ?si=u4A4-4AUqToEWCEl&t=198](https://youtu.be/tIazVdhsSqQ?si=u4A4-4AUqToEWCEl&t=198)

# <img src="/youtube.png" class="tab-icon"> YouTube Walkthrough

[https://youtu.be/7VrU\_zNrzqo](https://youtu.be/7VrU_zNrzqo)