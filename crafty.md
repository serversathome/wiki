---
title: Crafty Controller
description: A guide to deploying Crafty Controller
published: true
date: 2025-10-01T17:20:49.848Z
tags: 
editor: markdown
dateCreated: 2025-10-01T17:20:49.848Z
---

# ![](/crafty-controller.png){class="tab-icon"} What is Crafty Controller?

Vaultwarden is a free and secure password manager that works on any device and platform. It is the community version of Bitwarden, with features like organizations, attachments, API, and more.

# 1 · Deploy Crafty Controller
# {.tabset}
## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Under **Network Configuration** add **Additional Ports** of `25560`
1. Set the **Storage Configuration** to *Host Path* for the Config Storage, Servers Storage, Logs Storage, Backups Storage, Import Storage


## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
    crafty:
        container_name: crafty_container
        image: registry.gitlab.com/crafty-controller/crafty-4:latest
        restart: unless-stopped
        environment:
            - TZ=America/New_York
        ports:
            - "8443:8443" # HTTPS
            - "8123:8123" # DYNMAP
            - "19132:19132/udp" # BEDROCK
            - "25500-25600:25500-25600" # MC SERV PORT RANGE
        volumes:
            - /mnt/tank/configs/crafty/backups:/crafty/backups
            - /mnt/tank/configs/crafty/logs:/crafty/logs
            - /mnt/tank/configs/crafty/servers:/crafty/servers
            - /mnt/tank/configs/crafty/config:/crafty/app/config
            - /mnt/tank/configs/crafty/import:/crafty/import
```