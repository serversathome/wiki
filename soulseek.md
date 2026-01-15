---
title: Soulseek
description: A guide to deploying the SLSKD daemon via docker compose
published: true
date: 2026-01-15T15:31:53.073Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:08:40.062Z
---

# ![Soulseek](/slskd.png){class="tab-icon"} What is Soulseek?

**Soulseek** is a legendary, ad‑free peer‑to‑peer network for sharing music and rare files.  The modern container **[slskd](https://github.com/slskd/slskd)** runs Soulseek as a lightweight server daemon you can access via web‑UI or mobile clients.

---

<details class="quickstart" open>
<summary><strong>🚀 Quick‑Start Checklist</strong></summary>

1. **Deploy container** (Docker Compose *or* Docker + VPN).
2. **Create account** in the desktop client *(needed once)* → username & password.
3. **Mount music** → `/music`;  set **Downloads** → `/music/downloads`, **Incomplete** → `/music/incomplete`.
4. Log in to **web‑UI** → `Settings → Options` → add Soulseek credentials.
5. *(Recommended)* Run behind **Gluetun / AirVPN** for privacy.

</details>

---

# 1 · Deploy slskd

# tabs {.tabset}

## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  slskd:
    image: slskd/slskd:latest
    container_name: slskd
    user: 568:568           # TrueNAS media UID:GID
    environment:
      SLSKD_REMOTE_CONFIGURATION: "true"   # enable web‑UI config
      SLSKD_SLSK_LISTEN_PORT: 50300        # incoming port
      # Optional paths (leave blank to use /downloads & /incomplete inside /app)
      SLSKD_DOWNLOADS_DIR: /music/downloads
      SLSKD_INCOMPLETE_DIR: /music/incomplete
    ports:
      - 5030:5030   # Web‑UI
      - 5031:5031   # WebSocket API
      - 50300:50300 # P2P listen
    volumes:
      - /mnt/tank/configs/soulseek:/app
      - /mnt/tank/media/music:/music
    restart: unless-stopped
```

### Permissions & Folder Structure {.is-success}

* **PUID / PGID** — match media owner (TrueNAS default **568:568**).
* **Volumes** — configs in `/mnt/tank/configs/soulseek`, library in `/mnt/tank/media/music`.

> ⚠️ Soulseek exposes your IP.  Consider the VPN stack below for safety. {.is-warning}

---

## <img src="/docker.png" class="tab-icon"> Docker Compose + Gluetun VPN

```yaml
services:
  gluetun:
    image: qmcgaw/gluetun
    container_name: gluetun
    cap_add: [ NET_ADMIN ]
    devices: [ "/dev/net/tun:/dev/net/tun" ]
    environment:
      VPN_SERVICE_PROVIDER: airvpn
      VPN_TYPE: wireguard
      WIREGUARD_PRIVATE_KEY: "<key>"
      SERVER_COUNTRIES: CA
      FIREWALL_VPN_INPUT_PORTS: 50300
    ports:
      - 5030:5030     # slskd web‑UI
      - 5031:5031
      - 50300:50300   # P2P port (UDP/TCP inside tunnel)
    restart: unless-stopped

  slskd:
    image: slskd/slskd:latest
    container_name: slskd
    network_mode: service:gluetun   # route through VPN
    environment:
      SLSKD_REMOTE_CONFIGURATION: "true"
      SLSKD_SLSK_LISTEN_PORT: 50300
      SLSKD_DOWNLOADS_DIR: /music/downloads
      SLSKD_INCOMPLETE_DIR: /music/incomplete
    volumes:
      - /mnt/tank/configs/soulseek:/app
      - /mnt/tank/media/music:/music
    restart: unless-stopped
```

---

# 2 · First‑Run Configuration

1. Open `http://SERVER_IP:5030` → login (**slskd / slskd**).
2. **System → Options → edit slskd.yml**:

   * `soulseek.username: YOUR_USER`
   * `soulseek.password: YOUR_PASS`
   * *(optional)* `soulseek.description: "Hi, enjoy the tunes!"`
3. **Save → Restart daemon**.  Watch logs until **Connected to server** appears.
4. Under **Shares → Directories** add `/music` or any sub‑folders you want to share.

---

# 3 · Advanced Tweaks

| Option                  | Recommended   | Why                            |
| ----------------------- | ------------- | ------------------------------ |
| **listen\_port\_range** | `50300-50310` | Avoid ISP‑blocked default 2234 |
| **max\_uploads**        | `3`           | Prevent bandwidth hogging      |
| **download\_slots**     | `6`           | Balanced performance           |
| **skip\_extensions**    | `.log,.nfo`   | Saves space                    |

---

# 4 · Troubleshooting

<details><summary><strong>Cannot connect / red banner</strong></summary>
  
- Soulseek credentials wrong → re‑enter & restart.  
- Check ISP blocks → run via VPN.
  
</details>

<details><summary><strong>Downloads stuck at 0 B/s</strong></summary>
  
- Inbound port closed. Forward **50300** on router *or* use VPN with port‑forwarding.  
- Peer offline; try other sources.
  
</details>

<details><summary><strong>Permission denied saving files</strong></summary>
  
Ensure `chown 568:568 /mnt/tank/media/music*` and container runs as `user: 568:568`.
  
</details>

## ✏️ Editors & Contributors

> **Special thanks to the following members for reviewing and polishing this guide**
> - Dayce
> - Scar13t

Feel free to open a pull‑request or ping us on Discord if you spot an inaccuracy!

---

# <img src="/patreon-light.png" class="tab-icon"> Video Guide

*Coming soon – follow our [YouTube](https://www.youtube.com/@ServersatHome).*

[⇧ Back to top](#what-is-soulseek){.back-top}
