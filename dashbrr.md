---
title: Dashbrr
description: A guide to deploying Dashbrr
published: true
date: 2026-10-31T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-10-31T12:00:00.000Z
---

# <img src="/dashbrr.png" class="tab-icon"> What is Dashbrr?

**Dashbrr** is a dashboard for your media server stack, built by the team behind autobrr. Instead of a page of bookmarks, every card is live: it shows service health, active streams, download queues, pending requests and available updates, all pushed to the browser in real time.

- Media servers: Plex and Jellyfin, with active sessions and version checks
- The \*arr stack: Sonarr, Radarr, Lidarr, Readarr, Whisparr, Bazarr and Prowlarr
- Requests and cleanup: Overseerr and Maintainerr
- Downloaders: autobrr, SABnzbd, NZBGet and qui (qBittorrent)
- Infrastructure: Tailscale, Uptime Kuma, Traefik, plus a general card for any JSON health endpoint
- Built-in login or OIDC single sign-on, and installable as a PWA

<div class="glance">
  <div><span>Port</span><b><code>8080</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/autobrr/dashbrr"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Dashbrr
```yaml
services:
  dashbrr:
    image: ghcr.io/autobrr/dashbrr:latest
    container_name: dashbrr
    restart: unless-stopped
    user: "568:568"
    ports:
      - 8080:8080
    environment:
      - DASHBRR__DB_TYPE=sqlite
      - DASHBRR__DB_PATH=/config/dashbrr.db
      - DASHBRR__LISTEN_ADDR=0.0.0.0:8080
      - DASHBRR__LOG_LEVEL=info
    volumes:
      - /mnt/tank/configs/dashbrr:/config
```

1. Create the `/mnt/tank/configs/dashbrr` dataset before deploying so the container can write its database
1. Deploy the stack
1. Open the web UI at `http://YOUR-SERVER-IP:8080`

> Port `8080` is also the default for qBittorrent and SABnzbd. If either already uses it, change the left side of the mapping, for example `8081:8080`.
{.is-warning}

> The project's own compose file runs a separate Postgres container. SQLite is plenty for a home dashboard, so this guide keeps it to a single container. If you would rather use Postgres, set `DASHBRR__DB_TYPE=postgres` along with `DASHBRR__DB_HOST`, `DASHBRR__DB_PORT`, `DASHBRR__DB_USER`, `DASHBRR__DB_PASSWORD` and `DASHBRR__DB_NAME`.
{.is-info}

# 2 · Configuration

## 2.1 Create Your Account
1. On the first visit Dashbrr shows a **Register** form. There are no default credentials
1. Pick a username (3 to 32 characters) and a password of at least 8 characters

> Registration closes as soon as the first user exists, so whoever opens the page first becomes the admin. Register right after deploying.
{.is-warning}

You can also create or reset the account from the TrueNAS shell:
```bash
docker exec -it dashbrr dashbrr user create USERNAME PASSWORD
docker exec -it dashbrr dashbrr user change-password USERNAME NEWPASSWORD
```

## 2.2 Add Services
1. Click **Add Service** and choose the service type
1. Enter the service URL (for example `http://YOUR-SERVER-IP:8989` for Sonarr) and its API key
1. Dashbrr runs a health check as soon as the card is saved
1. Drag the cards around to arrange your dashboard

> Most API keys live under **Settings → General** in each \*arr app. Plex uses your Plex token and Overseerr uses the key under **Settings → General**.
{.is-info}

## 2.3 OIDC Login (Optional)
To sign in through Authentik, Pocket ID or another OIDC provider, add these to the `environment` section and redeploy:

```yaml
      - DASHBRR__OIDC_ISSUER=https://auth.example.com/application/o/dashbrr/
      - DASHBRR__OIDC_CLIENT_ID=your-client-id
      - DASHBRR__OIDC_CLIENT_SECRET=your-client-secret
      - DASHBRR__OIDC_REDIRECT_URL=https://dashbrr.example.com/api/auth/oidc/callback
```

The login page then shows an OIDC button alongside the built-in login.
