---
title: Wizarr
description: A guide to deploying Wizarr
published: true
date: 2026-06-22T10:14:06.126Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:09:40.051Z
---

# <img src="/wizarr.png" class="tab-icon"> What is Wizarr?

**Wizarr** is an automatic user invitation and management system for your media servers. Instead of manually creating accounts and walking friends and family through setup, you generate a single invite link and share it — Wizarr automatically adds the user to your server and guides them through downloading apps, accessing your request system, joining Discord, and more.

It supports **Plex, Jellyfin, Emby, Audiobookshelf, Romm, Komga, and Kavita**, with multi-tiered invitations, time-limited memberships, customizable wizard steps, request system integration (Overseerr, Ombi, etc.), and optional SSO.



# <img src="/docker.png" class="tab-icon"> 1 · Deploy Wizarr
# {.tabset}
## Docker

```yaml
services:
  wizarr:
    image: ghcr.io/wizarrrr/wizarr:latest
    container_name: wizarr
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
      - DISABLE_BUILTIN_AUTH=false
    restart: unless-stopped
    ports:
      - "5690:5690"
    volumes:
      - /mnt/tank/configs/wizarr:/data
```


> Set `DISABLE_BUILTIN_AUTH=true` **only** if you are putting Wizarr behind a dedicated auth provider such as Authelia or Authentik. Leaving it enabled keeps Wizarr's own login active.
{.is-warning}

## TrueNAS

Wizarr is available in the **Community** train of the TrueNAS Apps catalog.

1. Navigate to **Apps** in the TrueNAS UI and click **Discover Apps**.
2. Search for **Wizarr** and click the widget, then **Install**.
3. Configure the following settings:
   - **Web Port**: `5690`
   - **Timezone**: set to your local TZ (e.g. `America/New_York`)
   - **Storage**: point the app's data volume to a host path such as `/mnt/tank/configs/wizarr`
4. Click **Install** and wait for the app to move from *Deploying* to *Running*.


> The TrueNAS catalog app runs as user/group **568** (`apps`), matching the standard Servers@Home permission convention — no extra UID/GID changes needed.
{.is-success}

# 2 · Configuration

## 2.1 Connect a Media Server

On first launch you'll create an admin account, then add a server connection. Wizarr needs the server URL and an API key / token for each media platform you connect:

| Platform | What Wizarr Needs |
|----------|-------------------|
| Plex | Plex account token |
| Jellyfin / Emby | Server URL + API key |
| Audiobookshelf | Server URL + API key |
| Komga / Kavita | Server URL + credentials |
| Romm | Server URL + API key |


## 2.2 Create an Invitation

Once a server is connected, head to **Invitations** and create an invite. Key options include:

- **Expiration** — how long the invite link stays valid.
- **Membership duration** — optionally time-limit the user's access.
- **Libraries** — scope which libraries the invited user can see.
- **Wizard steps** — choose the onboarding pages (app download guides, request system, Discord, etc.) the user walks through after accepting.

Share the generated link with your user. When they open it, Wizarr adds them to the server and runs them through your configured setup wizard.

## 2.3 Customize the Wizard

The onboarding flow is fully editable under the wizard/settings area. You can add your own HTML snippets, reorder steps, and tailor the guide so new users land exactly where you want them — for example, straight into your Overseerr instance to start requesting content.





# <img src="/patreon-light.png" class="tab-icon"> 3 · Video

[![](/2025-04-14-wizarr-share-your-plex-jellyfi-promo-card.png)](https://www.patreon.com/posts/wizarr-share-or-126460048)