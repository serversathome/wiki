---
title: Windows Docker
description: A guide to deploying a Windows OS in a docker container
published: true
date: 2025-08-26T20:44:03.479Z
tags: 
editor: markdown
dateCreated: 2025-08-26T20:38:44.030Z
---

# <img src="/microsoft-windows.png" class="tab-icon"> What is the Windows Docker Container?
The Windows 11 Operating System within a docker container.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Windows
```yaml
services:
  windows:
    image: ghcr.io/dockur/windows:latest
    container_name: WinApps
    environment:
      VERSION: "11"
      RAM_SIZE: "4G" 
      CPU_CORES: "4" 
      DISK_SIZE: "64G"
      USERNAME: "admin"
      PASSWORD: "admin"
      HOME: "" # Set path to Linux user home folder.
    ports:
      - 8006:8006
      - 3389:3389/tcp
      - 3389:3389/udp
    cap_add:
      - NET_ADMIN 
    stop_grace_period: 120s
    restart: on-failure
    volumes:
      - /mnt/tank/configs/windows:/storage
      # - ${HOME}:/shared # Mount Linux user home directory @ '\\host.lan\Data'.
      - ./oem:/oem 
    devices:
      - /dev/kvm
      - /dev/net/tun
```

