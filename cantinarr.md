---
title: Cantinarr
description: A guide to deploying Cantinarr
published: true
date: 2026-09-08T15:25:12.685Z
tags: 
editor: markdown
dateCreated: 2026-09-08T15:19:38.173Z
---

#  <img src="/cantinarr.png" class="tab-icon"> What is Cantinarr?

**Cantinarr** is a self-hosted discovery, request, and \*arr management front end for your whole media stack. Household members browse movies, TV, books, and music powered by TMDB and Trakt, tap request, and get a push notification when it lands. You keep Radarr, Sonarr, Chaptarr, Lidarr, the download clients, Tautulli, and Plex invites behind the admin side. When a download gets stuck, Cantinarr diagnoses the cause in plain English and proposes a fix you approve. It also exposes itself as an MCP server, so Claude or any other MCP client can search your library and make requests for you. The whole thing is one Go container with an embedded Flutter web app, SQLite inside, and companion iOS and Android apps in beta.

<div class="glance">
  <div><span>Port</span><b><code>8585</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/windoze95/cantinarr"><i class="mdi mdi-github"></i>Project</a>
    <a href="https://cantinarr.com"><i class="mdi mdi-book-open-variant"></i>Docs</a>
    <a href="https://apps.truenas.com/catalog/cantinarr_community/"><i class="mdi mdi-server"></i>TrueNAS app</a>
  </div>
</div>



# 1 · Deploy Cantinarr
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

```yaml
services:
  cantinarr:
    image: ghcr.io/windoze95/cantinarr:latest
    container_name: cantinarr
    environment:
      # Optional push notifications for the iOS and Android apps.
      # Setting the URL enables push; the server enrolls itself on first start.
      - CANTINARR_PUSH_GATEWAY_URL=https://push.cantinarr.com
      # Origin your arrs POST webhooks back to. Set this when Cantinarr sits
      # behind a reverse proxy or on a different network than the arrs.
      # - CANTINARR_PUBLIC_URL=http://cantinarr:8585
      - CANTINARR_MEDIA_ROOTS=/media
      - PUID=568
      - PGID=568
    restart: unless-stopped
    ports:
      - "8585:8585"
    volumes:
      - /mnt/tank/configs/cantinarr:/config
      - /mnt/tank/media:/media:ro
```


## <img src="/truenas.png" class="tab-icon"> TrueNAS

Cantinarr is in the **Community** train of the TrueNAS apps catalog.

1. Go to **Apps** in the TrueNAS UI and search for **Cantinarr**
2. Click **Install**
3. Under **Cantinarr Configuration**:
   - **Push Gateway URL**: leave `https://push.cantinarr.com` for push notifications, or clear it to disable push
   - **Arr Webhook Callback URL**: the address your Radarr, Sonarr, Chaptarr, and Lidarr containers can reach Cantinarr at, for example `http://192.168.1.10:30472`. Leave it blank on a simple single-box setup
4. Under **Network Configuration**, note the **WebUI Port**. TrueNAS defaults it to `30472`, not 8585
5. Under **Storage Configuration**, set **Config Storage** to a **Host Path** such as `/mnt/tank/configs/cantinarr` if you want it outside an ixVolume, so it lands in your normal snapshot and replication tasks
6. To enable completed-media downloads, add an **Additional Storage** entry pointing at your media dataset, tick **Read Only**, and set the mount path to `/media`. Then add an environment variable `CANTINARR_MEDIA_ROOTS` with the value `/media`
7. Click **Install** and open the app once it reports **Running**

> The app runs as user and group `568` (the `apps` account). If you use a host path for config, make sure that account owns it or the container will not start.
{.is-info}

# 2 · First run

## 2.1 Create the admin account

Open the web UI and the setup wizard walks you through creating the first admin. Discovery and search work immediately on Cantinarr's built-in TMDB key, so there is no signup to do before you can browse. Add your own TMDB read access token later under **Settings > Providers & Credentials** if you would rather use your own account.

The wizard is a live checklist built from what is actually configured, so every step opens the real settings screen and progress cannot go stale.

## 2.2 Connect your services

Everything is added from the admin UI. There are no config files and no environment variables for API keys.

| Service | Where | Notes |
|---------|-------|-------|
| Radarr / Sonarr | Settings > Add Instance | Movies and TV |
| Chaptarr | Settings > Add Instance | Books, granted per user |
| Lidarr | Settings > Add Instance | Music, granted per user |
| SABnzbd / qBittorrent / NZBGet / Transmission | Settings > Add Instance | Queue, history, speeds |
| Tautulli | Settings > Add Instance | Plex activity and stats |
| Trakt client ID | Settings > Providers & Credentials | Better discovery plus fallback ID bridging |




Two gotchas worth knowing up front:

- A container using `network_mode: container:gluetun` has no hostname of its own, so `http://chaptarr:8789` will never resolve. Point the instance URL at the VPN gateway container that publishes the port instead.
- SABnzbd rejects hostnames it does not recognise. Add the name to `host_whitelist` under **Config > Special**, or set the container hostname to match.

When you add an \*arr instance, Cantinarr installs its own authenticated webhook on that instance automatically, which is what makes imports, deletes, and manual adds show up in the app instantly instead of on a poll. Each instance's edit screen has a **Configure instant updates** button to repair it if the callback address changes.

## 2.3 Add your household

Users are passwordless by default. Generate a connect link from **Settings > Users**, send it to the person, and opening it on their device creates the account and signs them in permanently. The session ends only when you revoke the device or delete the user.

Optional controls worth turning on:

- **Approvals**, globally or per user, so requests land in a queue you approve from the push notification
- **Per-user default instances**, if you run several Radarr or Sonarr instances
- **Season and quality choice** per user, or lock everyone to a default quality profile
- **Plex invites**, one tap from the Users screen once your Plex account is linked, or fully automatic

# 3 · The interesting parts

## 3.1 Import Doctor

When a download stalls, Cantinarr explains why in plain English: sample file, un-extracted archive, "not an upgrade", unparseable file, remote path mapping problems, stalled torrent, permissions. Each diagnosis comes with one-click fixes, including force import with the candidate files shown, remove plus blocklist plus re-search, hand off to something like Unpackerr, or a rescan.

## 3.2 Remediation agent

Users tap **Report a problem** and the agent opens a quiet observation window first, giving Sonarr or Radarr a chance to retry on its own before anyone gets paged. Only a persistent problem goes into the supervised workflow, where you approve the proposed fix. Ticking **Always approve** on an approval arms a standing rule for that exact problem and fix pair, and the rule pauses itself the moment a fix fails.

Remediation always uses the admin's shared AI credential, never a user's personal key.

## 3.3 AI assistant and MCP

Each user can bring their own Anthropic, OpenAI, Gemini, or xAI key, or you can configure a shared provider and grant access per person. Every saved model and credential has to pass a small live test turn before Cantinarr will activate it.

The server also exposes an MCP endpoint at `/mcp` with OAuth discovery, browser and passkey login, and a per-tool on/off list under **Settings > AI Tools**. That means Claude Desktop, Claude Code, or any other MCP client can search your library, check availability, and file requests against your own server.

> MCP clients authenticate over inbound OAuth, which needs a secure context for passkeys. On a plain HTTP deployment, give the account a password under **Settings > Users** instead. Behind a reverse proxy, set `CANTINARR_OAUTH_ISSUER` to your external HTTPS origin and keep it stable, since changing it forces every MCP client to reconnect.
{.is-info}


# 4 · Environment variables

Everything below is optional. Credentials are managed in the UI, not here.

| Variable | Default | Description |
|----------|---------|-------------|
| `CANTINARR_PORT` | `8585` | HTTP listen port |
| `CANTINARR_SERVER_NAME` | `Cantinarr` | Display name shown in clients |
| `CANTINARR_PUBLIC_URL` | request origin | Origin the \*arrs POST webhooks back to. Must be reachable **from the \*arr containers** |
| `CANTINARR_OAUTH_ISSUER` | request origin | External HTTPS origin for inbound MCP OAuth. Set it behind a proxy and keep it stable |
| `CANTINARR_ENCRYPTION_KEY` | auto-generated | Base64 32-byte key for secrets at rest. Defaults to `/config/encryption.key` |
| `CANTINARR_MEDIA_ROOTS` | unset | Comma-separated absolute paths allowed for completed-media downloads. Empty disables the feature |
| `CANTINARR_PUSH_GATEWAY_URL` | unset | Setting it enables push notifications and auto-enrolls on first start |
| `CANTINARR_DISABLE_UPDATE_CHECK` | unset | Set to `1` to turn off the periodic GitHub release check |
{.dense}

# 5 · Mobile apps

The native apps are in beta:

- **iPhone and iPad**: open public beta on [TestFlight](https://testflight.apple.com/join/bCPDwCsD)
- **Android**: closed testing, testers added by hand through the [project site](https://cantinarr.com/#android-beta)

Both talk only to your own server, so stand the container up first.

# <img src="/youtube.png" class="tab-icon"> 6 · Video

