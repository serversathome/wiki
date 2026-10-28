---
title: DropSync
description: A guide to deploying DropSync
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-03-21T11:26:27.867Z
---

# What is DropSync?

**DropSync** is a peer-to-peer file sharing application that sends files directly between two browsers using WebRTC — no accounts, no cloud storage, no file size limits. Once both browsers connect, the signaling server steps out of the way and everything flows directly between devices. Rooms can be password-protected with AES-GCM 256-bit end-to-end encryption, and the app includes built-in encrypted chat over the same P2P channel.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy DropSync


> DropSync publishes a prebuilt image on GitHub Container Registry. Only the `main` tag exists (there is no `latest` tag).
{.is-info}

```yaml
services:
  dropsync:
    image: ghcr.io/ivucicev/dropsync:main
    container_name: dropsync
    environment:
      - NODE_ENV=production
    restart: unless-stopped
    ports:
      - "3000:3000"
```

To use your own TURN server, build the image from source instead, because the `VITE_TURN_*` values are baked in at build time. Clone the repo and create a `.env.production` file in it (not `.env`, which the repo's `.dockerignore` keeps out of Docker builds):

```bash
git clone https://github.com/ivucicev/DropSync.git /mnt/tank/configs/dropsync/src
cat > /mnt/tank/configs/dropsync/src/.env.production <<'END'
VITE_TURN_URL=turn.yourdomain.com
VITE_TURN_USERNAME=dropsync
VITE_TURN_CREDENTIAL=your_turn_password
END
```

Then replace the `image:` line in the compose above with:

```yaml
    build: /mnt/tank/configs/dropsync/src
```

> DropSync uses WebRTC for direct peer-to-peer transfers. This works well between desktop browsers on standard home/broadband networks, but mobile networks (5G/LTE) use Carrier-Grade NAT (CGNAT). The image falls back to the free, rate-limited Open Relay TURN server; for reliable 5G/LTE use, build with your own TURN server (`VITE_TURN_*` values above).
{.is-warning}

# 2 · Configuration

## 2.1 How It Works

DropSync uses a simple three-step flow:

1. **Create a room** — Click **Create Secure Room** on the landing page. A unique room ID is generated. Optionally set a password to enable AES-GCM 256 end-to-end encryption for all files and chat in that room.
2. **Share the link** — Copy the room URL or scan the QR code. Send it to your peer however you like. If you set a password, they will be prompted for it before connecting.
3. **Drop files** — Once connected (status turns green), drag files onto the drop zone or tap to select. Your peer accepts the incoming file request and the download begins — directly from browser to browser.

Under the hood:
- Both browsers connect to the signaling server via Socket.IO
- WebRTC offer/answer and ICE candidates are exchanged
- A direct peer-to-peer `RTCDataChannel` is established
- The signaling server is now completely out of the loop
- Files are split into 8KB chunks, optionally encrypted, and streamed over the data channel



## 2.2 Cloudflare Tunnel

If you expose DropSync through a Cloudflare Tunnel, two settings need to be configured in the dashboard or the signaling connection will fail silently.

**1. Enable WebSockets**

In the main Cloudflare dashboard (not Zero Trust), go to your domain → **Network** → toggle on **WebSockets**. DropSync uses Socket.IO for the initial WebRTC handshake, which requires WebSocket support.

**2. Disable HTTP/2 connection to origin**

In the Zero Trust dashboard, go to **Networks** → **Tunnels** → select your tunnel → **Public Hostname** → edit the DropSync hostname → expand **Additional application settings** → toggle off **HTTP/2 connection**. Cloudflare's HTTP/2 multiplexing breaks WebSocket upgrades from the tunnel to your origin.
