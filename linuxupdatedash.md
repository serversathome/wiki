---
title: Linux Update Dashboard
description: A guide to deploying Linux Update Dashboard
published: true
date: '2026-11-04T23:32:08.140Z'
tags: []
editor: markdown
dateCreated: '2026-03-02T21:33:57.457Z'
---

# What is Linux Update Dashboard?

**Linux Update Dashboard** is a self-hosted web app for managing Linux package updates across multiple servers from a single interface. It connects to your servers via SSH, checks for available updates, and lets you apply them individually or in bulk — all from your browser. It supports APT, DNF, YUM, Pacman, Flatpak, and Snap with automatic package manager detection, and includes SSH-safe upgrades that survive connection drops via `nohup`. The app features encrypted SSH credentials (AES-256-GCM), four authentication methods (password, Passkeys, OIDC SSO, API tokens), flexible notifications (Email/SMTP, Gotify, MQTT, ntfy.sh, Telegram and webhooks), and a lightweight SQLite backend.

# :logos:docker-icon: 1 · Deploy Linux Update Dashboard


```yaml
services:
  dashboard:
    image: ghcr.io/theduffman85/linux-update-dashboard:latest
    container_name: linux-update-dashboard
    restart: unless-stopped
    ports:
      - "3001:3001"
    volumes:
      - /mnt/tank/configs/ludash/data:/data
    environment:
      - LUDASH_ENCRYPTION_KEY=
      - LUDASH_DB_PATH=/data/dashboard.db
      - NODE_ENV=production
```

Create the required encryption key:

```bash
openssl rand -base64 32
```


# 2 · Configuration


## 2.1 Adding Systems

::block-steps
1. Click **Add System** in the dashboard
2. Enter the SSH connection details (hostname, port, username)
3. Choose authentication method: password or SSH key
4. Click **Test Connection** — package managers and system info are detected automatically
5. Save the system
::

> [!TIP]
> SSH credentials are encrypted at rest using AES-256-GCM with per-entry random IVs. The credentials are never stored in plain text.

## 2.2 Reverse Proxy Setup

If running behind a reverse proxy, set the following environment variables:

```yaml
environment:
  - LUDASH_BASE_URL=https://ludash.yourdomain.com
  - LUDASH_TRUST_PROXY=true
```

**Caddy example:**

```
ludash.yourdomain.com {
    reverse_proxy localhost:3001
}
```

## 2.3 Authentication

Linux Update Dashboard supports four authentication methods that can be used simultaneously:

**Password** — Standard username/password with bcrypt hashing and 30-day JWT sessions. Can be disabled once a passkey or SSO provider is configured.

**Passkeys (WebAuthn)** — Register hardware keys or platform authenticators (Touch ID, Windows Hello) for passwordless login.

**SSO (OpenID Connect)** — Connect any OIDC-compatible identity provider (Authentik, Keycloak, Okta, etc.). Set the callback URL in your provider to `{LUDASH_BASE_URL}/api/auth/oidc/callback`.

**API Tokens** — Bearer tokens for external integrations like Homepage widgets or scripts. Create them from the Settings page with configurable permissions and expiry.

## 2.4 Notifications

Configure notification channels from the dashboard UI under the Notifications section. These channel types are supported:

- **Email/SMTP** — Send alerts via any SMTP server
- **Gotify** — Push notifications to a self-hosted Gotify server
- **MQTT** — Publish events to an MQTT broker
- **ntfy.sh** — Push notifications via ntfy
- **Telegram** — Messages through a Telegram bot
- **Webhook** — Custom HTTP requests for n8n, Node-RED and similar tools (includes a Discord preset)

Each channel can be scoped to specific systems and configured to trigger on specific event types. Notification digests can be scheduled using cron expressions for batched summaries instead of immediate alerts.


