---
title: Homarr
description: A guide to deploying Homarr
published: true
date: 2025-09-30T18:24:20.087Z
tags: 
editor: markdown
dateCreated: 2025-09-30T18:22:23.612Z
---

# ![](/homarr.png){class="tab-icon"} What is Homarr?
A sleek, modern dashboard that puts all of your apps and services at your fingertips. Control everything in one convenient location. Seamlessly integrates with the apps you've added, providing you with valuable information.


# 1 · Deploy Homarr
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  homarr:
    container_name: homarr
    image: ghcr.io/homarr-labs/homarr:latest
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/tank/configs/homarr:/appdata
    environment:
      - SECRET_ENCRYPTION_KEY=b811812a4982ee815bc30a5fb95c999912005d90fca73eafcd3e8758a09b298f
    ports:
      - 7575:7575
```

## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Set the **Secret Encryption Key** to something strong
1. Check the box to **Mount Docker Socket**
1. Set the **Homarr Data Storage** to *Host Path* and select a dataset to store the config files