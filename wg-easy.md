---
title: wg-easy
description: Configuring the wg-easy container to manage wireguard
published: true
date: 2025-11-19T11:06:30.190Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:39:17.982Z
---

# ![](/wireguard.png){class="tab-icon"} What is wg-easy?

wg-easy is the easiest way to run WireGuard VPN + Web-based Admin UI.


# 1 · Deploy wg-easy
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  wg-easy:
    environment:
    #  Optional:
    #  - PORT=51821
    #  - HOST=0.0.0.0
      - INSECURE=true

    image: ghcr.io/wg-easy/wg-easy:15
    container_name: wg-easy
    networks:
      wg:
        ipv4_address: 10.42.42.42
        ipv6_address: fdcc:ad94:bacf:61a3::2a
    volumes:
      - /mnt/tank/configs/wgeasy/:/etc/wireguard
      - /lib/modules:/lib/modules:ro
    ports:
      - "51820:51820/udp"
      - "51821:51821/tcp"
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1
      - net.ipv6.conf.all.disable_ipv6=0
      - net.ipv6.conf.all.forwarding=1
      - net.ipv6.conf.default.forwarding=1

networks:
  wg:
    driver: bridge
    enable_ipv6: true
    ipam:
      driver: default
      config:
        - subnet: 10.42.42.0/24
        - subnet: fdcc:ad94:bacf:61a3::/64
```


## <img src="/truenas.png" class="tab-icon"> TrueNAS

![](https://wiki.hydrology.cc/screen_shot_2023-12-09_at_7.58.29_am.png)

1. Change the **WG Easy Config Storage** to *Host Path*

# 2 · Settings

## 2.1 

# <img src="/youtube.png" class="tab-icon"> 2 · Video

