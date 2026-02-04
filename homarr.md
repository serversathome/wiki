---
title: Homarr
description: A guide to deploying Homarr
published: true
date: 2026-02-04T11:00:33.217Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:05:10.051Z
---

# <img src="/homarr.png" class="tab-icon"> What is Homarr?

**Homarr** is a modern, sleek dashboard for your homelab that puts all of your apps and services at your fingertips. It features a drag-and-drop interface with no YAML configuration required, making it accessible for beginners while remaining powerful enough for advanced users.

Key features include:

- Easy drag-and-drop dashboard customization
- 14+ built-in integrations (Sonarr, Radarr, Plex, Jellyfin, qBittorrent, and more)
- 10,000+ icons available from multiple repositories
- Built-in authentication and authorization (credentials, OIDC, LDAP)
- 26 languages supported
- Docker integration for container management
- Weather, calendar, and system monitoring widgets

# 1 · Deploy Homarr
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker

```yaml
services:
  homarr:
    image: ghcr.io/homarr-labs/homarr:latest
    container_name: homarr
    restart: unless-stopped
    ports:
      - "7575:7575"
    environment:
      - SECRET_ENCRYPTION_KEY=your_64_character_hex_string
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /mnt/tank/configs/homarr/appdata:/appdata
```

1. Generate a secret encryption key using `openssl rand -hex 32`
2. Replace `your_64_character_hex_string` with your generated key
 
> The Docker socket mount is optional but required for Docker integration features. If you don't need container management from Homarr, you can remove the `/var/run/docker.sock` volume.
{.is-info}



## <img src="/truenas.png" class="tab-icon"> TrueNAS

1. Navigate to **Apps** in the TrueNAS UI
1. Search for "**Homarr**" (Community train)
1. Click **Install**
1. Configure the following settings:
   - **Secret Encryption Key**: Generate with `openssl rand -hex 32`
   - **WebUI Port**: `7575` (default)
   - **Host Path for Appdata**: `/mnt/tank/configs/homarr`
   - **Docker Socket**: Enable if you want Docker integration
1. Click **Save**



# 2 · Configuration

## 2.1 Initial Setup

On first launch, you'll be guided through the setup wizard:

1. **Select Language** - Choose from 26 available languages
1. **Choose Theme** - Light or dark mode
1. **Create User** - Set up your admin username and password
1. **Privacy Settings** - Configure analytics, crawling, and indexing preferences
1. **Create Board** - Start building your first dashboard

## 2.2 Adding Apps

To add applications to your dashboard:

1. Click the **Edit** icon in the top-right corner
1. Click the **+** button to add a new item
1. Select **App** from the widget types
1. Configure the app details:
   - **Name**: Display name for the app
   - **Internal URL**: The URL Homarr uses to communicate with the app (e.g., `http://192.168.1.100:8989` for Sonarr)
   - **External URL**: The URL you use to access the app in your browser
   - **Icon**: Search from 10,000+ built-in icons or provide a custom URL
   
1. For apps with integrations (Sonarr, Radarr, etc.), add your API key in the **Integration** section

## 2.3 Widgets

Homarr includes several built-in widgets:

| Widget | Description |
|--------|-------------|
| Weather | Current conditions and forecast |
| Calendar | iCal feed integration, Sonarr/Radarr calendars |
| System Info | CPU, RAM, disk usage via Dash. integration |
| Docker | Container status and management |
| Clock | Digital clock with timezone support |
| Iframe | Embed external pages |
| RSS | RSS feed reader |
{.dense}

## 2.4 Integrations

Homarr supports deep integration with many popular self-hosted applications:

- **Media Management**: Sonarr, Radarr, Lidarr, Readarr, Prowlarr
- **Media Servers**: Plex, Jellyfin, Overseerr, Jellyseerr
- **Download Clients**: qBittorrent, Transmission, Deluge, SABnzbd, NZBGet
- **DNS/Ad-blocking**: Pi-hole, AdGuard Home
- **System Monitoring**: Dash.
- **Home Automation**: Home Assistant
- **Other**: TrueNAS, OpenMediaVault, Docker

To configure an integration:

1. Add the app to your dashboard
1. Click **Edit** on the app tile
1. Navigate to the **Integration** tab
1. Enter your API key and any required settings
1. Save and the widget will display live data

> 
> For integrations to work properly, use the internal IP address of your server rather than Docker internal hostnames, as clients outside the Docker network won't be able to resolve those names.
{.is-warning}


# <img src="/patreon-light.png" class="tab-icon"> 3 · Video

[![](/2025-06-03-build-the-ultimate-homarr-dashbo-promo-card.png)](https://www.patreon.com/posts/build-ultimate-130614993)