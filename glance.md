---
title: Glance
description: A guide to deploying Glance dashboard
published: true
date: 2025-11-04T14:38:18.641Z
tags: 
editor: markdown
dateCreated: 2025-11-04T14:22:18.630Z
---

# <img src="/glance.png" class="tab-icon"> What is Glance?

A lightweight, highly customizable dashboard that displays your feeds in a beautiful, streamlined interface.
# <img src="/docker.png" class="tab-icon"> 1 · Deploy Glance
```yaml
services:
  glance:
    container_name: glance
    image: glanceapp/glance
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/glance/:/app/config
    ports:
      - 8080:8080
```
## Sample config.yml
1. Navigate to your `/mnt/tank/configs/glance` directory
1. Run the following command **as root**:
    ```bash
    wget https://raw.githubusercontent.com/glanceapp/glance/refs/heads/main/docs/glance.yml
    ```
1. Start the container