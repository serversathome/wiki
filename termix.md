---
title: Termix
description: A guide to deploying Termix
published: true
date: 2026-05-06T22:04:27.210Z
tags: 
editor: markdown
dateCreated: 2026-05-06T22:04:27.210Z
---

# <img src="/termix.png" class="tab-icon"> What is Termix?

**Termix** is a free, open-source, self-hosted server management platform that puts SSH, RDP, VNC, and Telnet access into a single web interface. Think of it as a self-hosted alternative to Termius, with cross-device sync and no subscription. Beyond remote terminals and desktops, Termix includes SSH tunnel management with auto-reconnect, a remote file manager with sudo support, Docker container management, server stats dashboards, command snippets, and a network graph view of your homelab. It supports jump hosts, SOCKS5 proxies, OIDC, 2FA, and host key verification, and stores all data in an encrypted SQLite database. Available as a web app, desktop app for Windows/Linux/macOS, PWA, and dedicated mobile/tablet apps for iOS and Android.

# 1 · Deploy Termix
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

```yaml
services:
  termix:
    image: ghcr.io/lukegus/termix:latest
    container_name: termix
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - /mnt/tank/configs/termix:/app/data
    environment:
      PORT: "8080"

  guacd:
    image: guacamole/guacd:1.6.0
    container_name: guacd
    restart: unless-stopped

```

The `guacd` sidecar is required for RDP, VNC, and Telnet support. If you only plan to use SSH, you can omit `guacd`.


> Pin the `guacd` tag to a specific version (`1.6.0` shown above). The Guacamole protocol can change between major versions and Termix is built against a specific guacd release — using `:latest` can silently break remote desktop connections after a guacd upgrade.
{.is-warning}

## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Navigate to **Apps** in the TrueNAS UI
2. Search for "Termix" in the **Community** train
3. Click **Install**
4. Configure the following settings:
   - **Web Port**: `8080` (or your preferred port)
   - **Storage → App Data**: Host Path → `/mnt/tank/configs/termix`
   - **Network → Guacamole Daemon**: leave enabled to support RDP/VNC/Telnet
5. Click **Install**

> 
> The TrueNAS app runs Termix as UID/GID 1000 and `guacd` as UID/GID 568. Make sure `/mnt/tank/configs/termix` is writable by both, or use the dataset Apps preset and set ownership to UID 1000.
{.is-info}

# 2 · Initial Setup

## 2.1 Create Admin Account

Open `http://your-server:8080` in your browser. The first user you register becomes the admin account — there is no default username or password.

> 
> Register the admin account immediately after deployment. Until you do, the registration endpoint is open and anyone with network access can claim admin.
{.is-danger}


# 3 · Adding Hosts


Click **+ New Host** in the sidebar and pick the connection type. 

For RDP, set **Security mode** to `any` as a default — it lets `guacd` negotiate NLA, RDP, or TLS automatically. If a host fails to connect, switch to a specific mode like `nla` or `rdp`. Enable **Ignore Cert** for any RDP host using a self-signed certificate (most do).


# 4 · Remote Desktop Setup

## 4.1 Windows RDP Host Prep

For each Windows machine you want to RDP into:

1. **Settings → System → Remote Desktop** → toggle on (requires Pro, Enterprise, or Education edition)
2. Leave **Network Level Authentication** enabled — `guacd` handles NLA
3. Confirm the user account has a password set (RDP rejects blank passwords by default)
4. Set **Power & battery → Plugged in: Never sleep** to prevent the machine from dropping the connection
5. Reserve the IP in your DHCP server so it doesn't shift

## 4.2 Linux VNC Host Prep

Most modern Linux desktops include a VNC server. For GNOME, enable **Settings → Sharing → Remote Desktop**. For headless servers, install `tigervnc-standalone-server` or `x11vnc` and run on display `:0` for an existing session, or `:1` for a new virtual display.


# <img src="/youtube.png" class="tab-icon"> 5 · Video
