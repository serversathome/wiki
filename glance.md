---
title: Glance
description: A guide to deploying Glance dashboard
published: true
date: '2026-11-04T23:31:24.465Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:04:49.727Z'
---

# What is Glance?

A lightweight, highly customizable dashboard that displays your feeds in a beautiful, streamlined interface.

# :logos:docker-icon: 1 · Deploy Glance
```yaml
services:
  glance:
    container_name: glance
    image: glanceapp/glance
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/glance/:/app/config
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - 8080:8080
```
## Sample config.yml

::block-steps
1. Navigate to your `/mnt/tank/configs/glance` directory
1. Run the following command **as root**:
    ```bash
    wget https://raw.githubusercontent.com/glanceapp/glance/refs/heads/main/docs/glance.yml
    ```
1. Start the container
::

> [!NOTE]
> When editing the glance.yml file, you don't need to restart the container for the changes to take effect, this happens automatically when saving the file.

> [!CAUTION]
> If you don't see your changes, check the logs to see what is wrong with the config file.

