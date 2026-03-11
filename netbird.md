---
title: Netbird
description: A guide to installing and using Netbird
published: true
date: 2026-03-11T14:58:33.794Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:06:37.607Z
---

# ![](/netbird.png){class="tab-icon"} What is Netbird?
NetBird combines a WireGuard®-based overlay network with Zero Trust Network Access, providing a unified open-source platform for reliable and secure connectivity

# 1 · Deploy Netbird
You must first go to [https://netbird.io](https://netbird.io) and click Get Started. After you create an account:
1. Navigate to **Setup Keys** ➡ **Create Setup Keys**
1. Give it a name
1. Switch the Reusable to ON
1. Set the expiry to `0`

# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose
```yaml
services:
  netbird:
    cap_add:
      - NET_ADMIN
    container_name: netbird
    environment:
      - NB_SETUP_KEY=
    image: netbirdio/netbird:latest
    network_mode: host
    restart: unless-stopped
    volumes:
      - ./netbird-client:/etc/netbird
```
Paste your Setup Key created from above in the `- NB_SETUP_KEY=` line.

## <img src="/linux.png" class="tab-icon"> Bare Metal Linux

```bash
curl -fsSL https://pkgs.netbird.io/install.sh | sh
```
Once this is installed, execute `netbird up --setup-key <key-value>` to start the connection using the setup key created above.

## <img src="/microsoft-windows.png" class="tab-icon"> Windows
1. Download the installer from [https://pkgs.netbird.io/windows/x64](https://pkgs.netbird.io/windows/x64)
1. Click on "Connect" from the NetBird icon in your system tray
a. Alternatively you can connect through the command line by executing `netbird up --setup-key <key-value>` using the setup key created above

## Mobile

Get the app from your respective app store:
- [Android *Google Play Store*](https://play.google.com/store/apps/details?id=io.netbird.client)
- [iOS *Apple App Store*](https://apps.apple.com/us/app/netbird-p2p-vpn/id6469329339)
{.links-list}

# 2 · Reverse Proxy

# <img src="/youtube.png" class="tab-icon"> 4 · Video
https://youtu.be/skbWnMSwZcE
