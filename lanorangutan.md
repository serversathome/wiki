---
title: LAN-Orangutan
description: A guide to deploying LAN-Orangutan
published: true
date: 2026-08-28T20:48:50.224Z
tags: 
editor: markdown
dateCreated: 2026-08-28T20:48:50.224Z
---

# <img src="/lan-orangutan.png" class="tab-icon"> What is LAN Orangutan?

**LAN Orangutan** is a lightweight, self-hosted network scanner from [291 Group](https://291group.com) that discovers the devices on your networks and lets you label, group, and track them over time. It wraps `nmap` in a clean web dashboard with a full CLI behind it, ships as a single Go binary with no runtime dependencies, and adds Tailscale integration so your tailnet peers show up alongside the machines on your LAN.



# <img src="/docker.png" class="tab-icon"> 1 · Deploy LAN Orangutan



```yaml
services:
  lan-orangutan:
    image: ghcr.io/291-group/lan-orangutan:latest
    container_name: lan-orangutan
    restart: unless-stopped
    network_mode: host
    cap_add:
      - NET_RAW
      - NET_ADMIN
      - NET_BIND_SERVICE
    environment:
      - ORANGUTAN_PORT=291
      - TZ=America/New_York
      # - ORANGUTAN_PASSWORD=change-me
    volumes:
      - /mnt/tank/configs/lan-orangutan:/var/lib/lan-orangutan
```

Open `http://<host-ip>:291` and create your password on first load




# 2 · Configuration

## 2.1 First Run

The first time the dashboard loads, LAN Orangutan asks you to create a password. Until you do, every page and every API endpoint is refused — there is no default password and none is generated for you. What you set is stored as a bcrypt hash in a file called `auth` inside the data directory, separate from your config file.

Sessions last a week by default, and five wrong guesses lock that IP out for fifteen minutes.

> **Locked out?** Delete the `auth` file in your data directory (`/mnt/tank/configs/lan-orangutan/auth` with the compose above) and restart. You are returned to the create-a-password screen; devices, labels, and settings are untouched. There is no email reset by design — the app has no account system and sends nothing off the machine.
{.is-info}


## 2.2 Scanning VLANs and Routed Subnets

LAN Orangutan finds networks by reading the host's own interfaces, so a VLAN or a subnet that only exists on the other side of a router never gets offered. List those explicitly:

```bash
ORANGUTAN_NETWORKS=192.168.10.0/24,10.0.5.0/24 orangutan serve
```

Or in the config file:

```ini
[scanning]
networks = 192.168.10.0/24, 10.0.5.0/24
```

They then appear on the dashboard next to the detected networks and scan the same way.

> Results for a routed network have no MAC addresses or manufacturer names — those come from ARP, which only works on the same segment. And this is **not** a workaround for Docker on macOS or Windows; the VM's NAT will still answer for addresses that do not exist.
{.is-warning}

# 3 · Dashboard

The web UI gives you:

- Real-time online/offline status per device
- Grouping by type (Server, Desktop, Laptop, Mobile, IoT, and so on)
- Labels and free-text notes on every device
- Search and filtering
- Export to CSV or JSON
- Continuous scanning you can toggle on and off to keep the view live
- Light/dark mode

Keyboard shortcuts: <kbd>/</kbd> to search, <kbd>R</kbd> to refresh, <kbd>T</kbd> to toggle the theme.

Scans run in the background, so the dashboard stays responsive and long scans do not time out. Progress shows the network being scanned, the device count so far, and a time estimate based on how long that network took last time — the very first scan of a network shows elapsed time instead, since there is nothing to estimate from. A large network takes a few minutes and you can cancel at any point.

# 4 · Tailscale

If Tailscale is connected on the host, its peers are pulled in automatically and added to your device list alongside everything found by scanning.

Peers can't be discovered by scanning at all — Tailscale gives every node its own single-address network, so there is no range to sweep. Reading them from Tailscale directly is faster anyway and needs no elevated privileges. Only currently-online peers are listed, shown with their tailnet hostname and OS, and they have no MAC address so no vendor lookup happens.

The dashboard and settings pages also show connection state with a connect/disconnect button, and surface the sign-in link if the machine still needs to authenticate.

> **Tailscale features need a host install.** They rely on the Tailscale CLI being present where LAN Orangutan runs. In the standard Docker setup the container has its own filesystem and cannot see the host's Tailscale, so the card reads "Not Installed" even when the host is on your tailnet. Run the binary on the host if you want this.
{.is-warning}

> Disconnecting warns you first — worth reading, since you may well be reaching the dashboard over Tailscale itself.
{.is-info}

