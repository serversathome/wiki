---
title: Windows Docker
description: A guide to deploying a Windows OS in a docker container
published: true
date: 2026-01-15T15:32:34.532Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:09:37.478Z
---

# <img src="/microsoft-windows.png" class="tab-icon"> What is the Windows Docker Container?
The Windows 11 Operating System within a docker container.

<div class="glance">
  <div><span>Port</span><b><code>8006</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://youtu.be/JtVfJn2drY4"><i class="mdi mdi-youtube"></i>Video walkthrough</a>
  </div>
</div>

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

# <img src="/youtube.png" class="tab-icon"> 2 · Video 
https://youtu.be/JtVfJn2drY4