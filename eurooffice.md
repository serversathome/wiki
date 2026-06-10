---
title: Euro-Office
description: A guide to deploying Euro-Office in docker
published: true
date: 2026-06-10T12:19:16.450Z
tags: 
editor: markdown
dateCreated: 2026-06-09T11:51:24.911Z
---

# What is Euro-Office?

**Euro-Office** is a self-hosted, web-based office suite for documents, spreadsheets, presentations, and PDFs, with real-time collaborative editing. It is backed by a coalition of European companies (IONOS, Nextcloud, and others) and pitched as a sovereign, fully open-source alternative to Microsoft 365 and Google Workspace, licensed under the AGPL.

Under the hood, Euro-Office is a fork of **OnlyOffice**. On launch it is functionally the OnlyOffice Document Server — rebranded, with a modernised build system and the first commits of its own ODF-focused work. The piece that runs today is the **Document Server** (the editing engine). It is not designed for standalone use: it is meant to be embedded in a host application (a file-sharing platform, wiki, or project tool). 


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Euro-Office


```yaml
services:
  euro-office:
    image: ghcr.io/euro-office/documentserver:latest
    container_name: euro-office
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      - JWT_ENABLED=true
      - JWT_SECRET=
    volumes:
      - /mnt/tank/configs/euro-office/data:/var/www/onlyoffice/Data
      - /mnt/tank/configs/euro-office/logs:/var/log/onlyoffice
      - /mnt/tank/configs/euro-office/lib:/var/lib/onlyoffice
```

Generate a real JWT secret with <kbd>openssl rand -hex 32</kbd> and replace the placeholder.


# 2 · Configuration

## 2.1 Environment Variables

| Variable | Purpose |
|----------|---------|
| `JWT_ENABLED` | Require signed requests. Keep `true`. |
| `JWT_SECRET` | Shared secret the host app (Nextcloud) must match exactly. |
| `EXAMPLE_ENABLED` | Enables the built-in test app at `/example`. Use for testing only; disable in production. |


# 3 · Reverse Proxy & Reliability
 
The Document Server must be reachable by the **browser** over HTTPS at its own hostname, because the browser loads the editor directly from it. Pointing Nextcloud at a plain-HTTP `http://ip:8080` address will fail with a blank editor (mixed content). Set up a hostname such as `office.example.com`.
 
For a reliable connection, **all three** of these paths must work:
 
| Path | Why |
|------|-----|
| Browser → Document Server | Loads the editor UI and JavaScript |
| Nextcloud → Document Server | Health/command check on save |
| Document Server → Nextcloud | The save-back callback (WOPI). If this fails, edits never persist. |

 

# 4 · Connecting to Nextcloud
 
On **Nextcloud Hub 26 Spring** the office integration is chosen under **Apps → Office → Select your preferred office suite**, where Euro-Office appears as **Nextcloud Office (Powered by Euro-Office)** alongside Collabora.
 
1. Select **Nextcloud Office**.
2. Open the Euro-Office admin settings and set the **Document Server address** to your proxy hostname (e.g. `https://office.example.com/`) — use the public FQDN, not a LAN IP, or the connector will refuse to connect to a local address.
3. Enter the **Secret key (JWT)** — the exact `JWT_SECRET` from your compose — and save. You should see a green "server is reachable" confirmation.
Opening a `.docx`, `.xlsx`, or `.pptx` in Nextcloud Files will then launch the Euro-Office editor inline, with real-time collaboration.
 

 
# <img src="/youtube.png" class="tab-icon"> 5 · Video

