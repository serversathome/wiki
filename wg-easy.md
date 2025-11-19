---
title: wg-easy
description: Configuring the wg-easy container to manage wireguard
published: true
date: 2025-11-19T10:48:50.817Z
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

You will need to change some settings upon install.

1.  **Hostname or IP**: your static IP or a domain name you own which points to your home IP. By default this is your private IP which is useless since it is not reachable from the greater internet.
2.  **Password for WebUI**: change this
3.  **Client DNS Server**: I recommend changing this to 9.9.9.9
4.  **Allowed IP Entry**: By default there is nothing added yet. This represents a value of 0.0.0.0/0 which = all IPs, which is a *full tunnel*. A full tunnel means when you connect to Wireguard all of your traffic, no matter where it is going, will be routed through your TrueNAS server first, then go out to the greater internet. I don't use this, I have a full VPN to do that. Instead, I do a *split tunnel*. A split tunnel means only part of the traffic is routed through Wireguard; the rest skips it and goes out to the web as if the VPN wasn't connected. To do this, we need to tell wg-easy which IP addresses we want to go through the tunnel. Since my home network is a 192.168.1.x net, my entry for this line is **192.168.1.0/24** and **10.8.0.0/24**. This tells wg-easy if I type in a 192.168.1.x address, or try to connect to another PC on the wg-easy VPN (which are the 10.8.0.x addresses) to route those through the VPN. Everything else skips the VPN and just exits to the internet.
5.  **Additional Environment Variables**
> This section is optional and should only be used if you cannot VPN in and reach other endpoints not on the wireguard server
{.is-warning}


|  Variable   |  Value   |
| --- | --- |
| WG\_POST\_UP | iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o br0 -j MASQUERADE; iptables -A INPUT -p udp -m udp --dport 51820 -j ACCEPT; iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; |
| WG\_PRE\_UP | iptables -t nat -F; iptables -F; |

You need to do something before you add the command above. You will see the "br0" entry here. This is the name of my network interface. Yours is found in the **Network** tab in the **Interfaces** box, and should look something like enp3s0 or something similar. Replace my "br0" with whatever your "enp5s0" is.

6\. **Networking**: Change the top number to 51820 and the bottom number to 51821. This is standard for wireguard and will make your life 1000% easier for troubleshooting down the road. This means 51820 will be your port forward from your router and http://{truenasip}:51821 will be the address you use to add clients.

Once this is done, go to your **System Settings** > **Advanced** > **Sysctl** box and add these two items: 

| Variable    |  Value   |
| --- | --- |
| net.ipv4.ip\_forward | 1   |
| net.ipv4.conf.all.src\_valid\_mark | 1   |

Last step, traffic will go from wherever you are to the IP/domain you added during install. The issue is your router will block this once it gets there. You need to set up a **port forward** from your router to your TrueNAS IP in its settings. Since your router is different than mine I can't help you here, but Google how to do port forwarding for your router model and you'll find a YouTube video on how to do it.

# <img src="/youtube.png" class="tab-icon"> 2 · Video

[](https://youtu.be/aPF_JhuwKmQ)