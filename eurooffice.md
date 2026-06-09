---
title: Euro-Office
description: A guide to deploying Euro-Office in docker
published: true
date: 2026-06-09T11:51:24.911Z
tags: 
editor: markdown
dateCreated: 2026-06-09T11:51:24.911Z
---

# What is Euro-Office?

**Euro-Office** is a self-hosted, web-based office suite for documents, spreadsheets, presentations, and PDFs, with real-time collaborative editing. It is backed by a coalition of European companies (IONOS, Nextcloud, and others) and pitched as a sovereign, fully open-source alternative to Microsoft 365 and Google Workspace, licensed under the AGPL.

Under the hood, Euro-Office is a fork of **OnlyOffice**. On launch it is functionally the OnlyOffice Document Server — rebranded, with a modernised build system and the first commits of its own ODF-focused work. The piece that runs today is the **Document Server** (the editing engine). It is not designed for standalone use: it is meant to be embedded in a host application (a file-sharing platform, wiki, or project tool). 


> The desktop apps are not yet released (the repo has no published builds), and the Nextcloud App Store listing is not live yet — the Nextcloud connector is currently a build-from-source step. See section 4.
{.is-info}

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
      - EXAMPLE_ENABLED=true
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


## 2.2 Try the Editors (no host app needed)

With `EXAMPLE_ENABLED=true`, browse to:

```
http://<server-ip>:8080/example
```

The example app acts as a stand-in host so you can create and open documents directly against the engine. Create a blank DOCX, XLSX, or PPTX, or open an existing Microsoft file to test format compatibility. To see real-time collaboration, open the same document in a second browser window — you will see live cursors and changes sync between both.

> 
> The `/example` page is officially a testing and integration harness, not the intended production interface. For day-to-day use, connect the Document Server to a host application such as Nextcloud.
{.is-warning}

# 3 · Reverse Proxy

If you expose the Document Server through HTTPS (for example via a Cloudflare Tunnel), give it its own hostname (e.g. `office.example.com` → `http://euro-office:80`). If Nextcloud is served over HTTPS while the Document Server is plain HTTP on `:8080`, the browser can block the editor on mixed-content rules even though the server-to-server JWT check passes. Serving both over HTTPS avoids a blank editor pane.

# 4 · Connecting to Nextcloud

The dedicated Euro-Office app in the Nextcloud App Store is coming soon.


> The Document Server address must be reachable from both the user's browser and the Nextcloud server, and Nextcloud must be reachable from the Document Server for the save-back callback to work.
{.is-info}



# <img src="/youtube.png" class="tab-icon"> 5 · Video
