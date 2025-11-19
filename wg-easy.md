---
title: wg-easy
description: Configuring the wg-easy container to manage wireguard
published: true
date: 2025-11-19T11:46:33.601Z
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

1. Change the **WG Easy Config Storage** to *Host Path*

# 2 · Settings
Click **Administrator** in the top right, then select **Admin Panel**.

## 2.1 Config

1. Change the **Allowed IPs** if you would like to run a split tunnel. This field should accept a comma-separated list.

> A value of `0.0.0.0/0` which = all IPs is a *full tunnel*. A full tunnel means when you connect to Wireguard all of your traffic, no matter where it is going, will be routed through your TrueNAS server first, then go out to the greater internet. I don't use this, I have a full VPN to do that. Instead, I do a *split tunnel*. A split tunnel means only part of the traffic is routed through Wireguard; the rest skips it and goes out to the web as if the VPN wasn't connected. To do this, we need to tell wg-easy which IP addresses we want to go through the tunnel. Since my home network is a 192.168.1.x net, my entry for this line is `192.168.1.0/24,10.8.0.0/24`. This tells wg-easy if I type in a 192.168.1.x address, or try to connect to another PC on the wg-easy VPN (which are the 10.8.0.x addresses) to route those through the VPN. Everything else skips the VPN and just exits to the internet.
{.is-info}


2. Change the **DNS** fot IPv4 to `1.1.1.1`

## 2.2 Interface

1. Click the **Change CIDR** button at the bottom if you want to use something other than the `10.8.0.0/24` range.



## 2.3 Hooks
If you need to adjust the **Preup, Postup, Predown, or Postdown** this is where those options are. 

# <img src="/youtube.png" class="tab-icon"> 3 · Video

