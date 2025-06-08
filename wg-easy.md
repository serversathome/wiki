---
title: wg-easy
description: Configuring the wg-easy container to manage wireguard
published: true
date: 2025-06-08T18:39:22.014Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:39:17.982Z
---

![](/wireguard.png)

![](/screenshot_from_2024-02-23_11-04-33.png)

# What is wg-easy?

wg-easy is the easiest way to run WireGuard VPN + Web-based Admin UI.


# Installation
# {.tabset}
## Docker Compose

> wg-easy has recently been overhauled and this page does not reflect the new changes yet!
{.is-danger}


```yaml
services:
  wg-easy:
    image: ghcr.io/wg-easy/wg-easy
    restart: unless-stopped
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1
    cap_add:
      - SYS_MODULE
      - NET_ADMIN
    ports:
      - '51821:51821/tcp'
      - '51820:51820/udp'
    volumes:
      - './wg-easy:/etc/wireguard'
    environment:
      - WG_PORT=51820
      - PORT=51821
      - PASSWORD_HASH=<🚨YOUR_ADMIN_PASSWORD_HASH>
      - WG_HOST=<🚨YOUR_SERVER_IP>
      - LANG=en
      - WG_DEFAULT_ADDRESS=10.8.0.x
      - WG_DEFAULT_DNS=9.9.9.9
      - WG_MTU=1420
      - WG_PERSISTENT_KEEPALIVE=120
      - WG_ALLOWED_IPS=0.0.0.0/0
      - UI_TRAFFIC_STATS=true
      - UI_CHART_TYPE=1
     
    container_name: wg-easy
```

### Options

| Env | Default | Example | Description |
| --- | --- | --- | --- |
| `PORT` | `51821` | `6789` | TCP port for Web UI. |
| `WEBUI_HOST` | `0.0.0.0` | `localhost` | IP address web UI binds to. |
| `PASSWORD_HASH` | \-  | `$2y$05$Ci...` | When set, requires a password when logging in to the Web UI. See [How to generate an bcrypt hash.md](https://github.com/wg-easy/wg-easy/blob/master/How_to_generate_an_bcrypt_hash.md) to learn how to generate the hash.<br><br>(The easier way is to to go to [https://it-tools.tech/bcrypt](https://it-tools.tech/bcrypt) and create the hash there and replace any $ symbols with $$) |
| `WG_HOST` | \-  | `vpn.myserver.com` | The public hostname of your VPN server. |
| `WG_DEVICE` | `eth0` | `ens6f0` | Ethernet device the wireguard traffic should be forwarded through. |
| `WG_PORT` | `51820` | `12345` | The public UDP port of your VPN server. WireGuard will listen on that (othwise default) inside the Docker container. |
| `WG_CONFIG_PORT` | `51820` | `12345` | The UDP port used on [Home Assistant Plugin](https://github.com/adriy-be/homeassistant-addons-jdeath/tree/main/wgeasy) |
| `WG_MTU` | `null` | `1420` | The MTU the clients will use. Server uses default WG MTU. |
| `WG_PERSISTENT_KEEPALIVE` | `0` | `25` | Value in seconds to keep the "connection" open. If this value is 0, then connections won't be kept alive. |
| `WG_DEFAULT_ADDRESS` | `10.8.0.x` | `10.6.0.x` | Clients IP address range. |
| `WG_DEFAULT_DNS` | `1.1.1.1` | `8.8.8.8, 8.8.4.4` | DNS server clients will use. If set to blank value, clients will not use any DNS. |
| `WG_ALLOWED_IPS` | `0.0.0.0/0, ::/0` | `192.168.15.0/24, 10.0.1.0/24` | Allowed IPs clients will use. |
| `WG_PRE_UP` | `...` | \-  | See [config.js](https://github.com/wg-easy/wg-easy/blob/master/src/config.js#L19) for the default value. |
| `WG_POST_UP` | `...` | `iptables ...` | See [config.js](https://github.com/wg-easy/wg-easy/blob/master/src/config.js#L20) for the default value. |
| `WG_PRE_DOWN` | `...` | \-  | See [config.js](https://github.com/wg-easy/wg-easy/blob/master/src/config.js#L27) for the default value. |
| `WG_POST_DOWN` | `...` | `iptables ...` | See [config.js](https://github.com/wg-easy/wg-easy/blob/master/src/config.js#L28) for the default value. |
| `LANG` | `en` | `de` | Web UI language (Supports: en, ua, ru, tr, no, pl, fr, de, ca, es, ko, vi, nl, is, pt, chs, cht, it, th, hi). |
| `UI_TRAFFIC_STATS` | `false` | `true` | Enable detailed RX / TX client stats in Web UI |
| `UI_CHART_TYPE` | `0` | `1` | UI\_CHART\_TYPE=0 # Charts disabled, UI\_CHART\_TYPE=1 # Line chart, UI\_CHART\_TYPE=2 # Area chart, UI\_CHART\_TYPE=3 # Bar chart |

## TrueNAS

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

# YouTube Walkthrough

https://youtu.be/aPF_JhuwKmQ