---
title: Dockge
description: A guide to installing Dockge on Ubuntu Server LTS
published: true
date: 2026-10-18T09:49:50.183Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:02:22.797Z
---


# ![](/dockge.png){class="tab-icon"} What is Dockge?

Dockge is a self-hosted Docker stack manager developed by the same person behind the popular software Uptime Kuma.

This software allows you to manage multiple Docker compose files from a single, easy-to-use interface. It is similar to the stack system Portainer implements but cleaner and simpler to use.

<div class="glance">
  <div><span>Port</span><b><code>5001</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://youtu.be/mVbi6xkM-rk"><i class="mdi mdi-youtube"></i>Video walkthrough</a>
  </div>
</div>

# 1 · Deploy Dockge
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS

![](/screenshot_from_2024-11-08_11-34-31.png)

1. Change the **WebUI Port** to **5001** since that is the default normally used by Dockge.
1. Change the **Dockge Stacks Storage** to **Hostpath** and make sure the dataset has permissions set for the **apps** user. 

## <img src="/docker.png" class="tab-icon"> Docker

> Note that Docker has to be installed before you can follow these steps!
{.is-warning}


### Docker Run

```bash
# Create directories that store your stacks and stores Dockge's stack
mkdir -p /opt/stacks /opt/dockge
cd /opt/dockge

# Download the compose.yaml
curl https://raw.githubusercontent.com/louislam/dockge/master/compose.yaml --output compose.yaml

# Start the server
docker compose up -d

# If you are using docker-compose V1 or Podman
# docker-compose up -d
```

### Docker Compose

```yaml
services:
 dockge:
   image: louislam/dockge:latest
   restart: unless-stopped
   container_name: dockge
   ports:
     - 5001:5001
   volumes:
     - /var/run/docker.sock:/var/run/docker.sock
     - /mnt/tank/configs/dockge:/app/data
     - /mnt/tank/stacks:/mnt/tank/stacks
   environment:
     - DOCKGE_STACKS_DIR=/mnt/tank/stacks
     - PUID=568
     - PGID=568
     - DOCKGE_ENABLE_CONSOLE=true
```

# 2 · Logging In

Navigate to http://{serverIP}:5001 and create a user and password to login.

# <img src="/youtube.png" class="tab-icon"> 3 · Video

[https://youtu.be/LpAxsO7zAMA](https://youtu.be/mVbi6xkM-rk)