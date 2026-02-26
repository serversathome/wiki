---
title: Cinephage
description: A guide to deploying Cinephage
published: true
date: 2026-02-26T16:28:22.385Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:39.889Z
---

# <img src="/cinephage.png" class="tab-icon"> What is Cinephage?

**Cinephage** is an all-in-one, self-hosted media management application that consolidates the functionality of Radarr, Sonarr, Prowlarr, and Bazarr into a single unified platform. The name comes from the Greek *cine* (film) + *phage* (to devour) — a film devourer.

Instead of juggling multiple *arr apps with their own databases and configurations, Cinephage gives you one database, one interface, and one configuration to manage your entire media workflow. It features built-in indexer support (20+), smart quality scoring with 100+ format attributes, automated subtitle fetching from 8 providers in 80+ languages, and a modern UI built with SvelteKit 5 and TailwindCSS 4.

Key highlights include `.strm` file support for streaming without downloading, Torznab/Newznab support for external indexer integration, support for multiple download clients (qBittorrent, SABnzbd, NZBGet, NZBMount), and automated monitoring for missing content and quality upgrades.

> 
> Cinephage is currently a **work in progress**. Early adopters are welcome, but expect some rough edges. Check the [GitHub repo](https://github.com/MoldyTaint/Cinephage) for the latest status.
{.is-warning}

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Cinephage

```yaml
services:
  cinephage:
    image: ghcr.io/moldytaint/cinephage:latest
    container_name: cinephage
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    ports:
      - "3000:3000"
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
      - ORIGIN=http://10.99.0.242:3000
    volumes:
      - /mnt/tank/configs/cinephage:/config
      - /mnt/tank/media:/media
```
> 
> Both Cinephage and your download client must mount the **same download directory** for imports to work correctly.
{.is-warning}


# 2 · Configuration

## 2.1 Initial Setup (Setup Wizard)

On first launch, Cinephage will walk you through a setup wizard:

1. **TMDB API Key** — Get a free API key at [themoviedb.org/settings/api](https://www.themoviedb.org/settings/api). This is used for all metadata, artwork, and content discovery.
2. **Download Client** — Configure your download client connection (qBittorrent, SABnzbd, NZBGet, or NZBMount). Make sure the download directory matches what you mounted in the compose file.
3. **Root Folders** — Set up your root folders for movies and TV shows (e.g., `/media/movies` and `/media/tv`).
4. **Indexers** — Enable the built-in indexers you want to use, or add Torznab/Newznab endpoints for Prowlarr/Jackett integration.

## 2.2 Quality Profiles

Cinephage includes 4 built-in quality profiles that score releases across 100+ format attributes:

| Profile | Description |
|---------|-------------|
| **Best** | Highest quality — prefers remuxes, lossless audio, HDR |
| **Efficient** | Good balance of quality and file size |
| **Micro** | Smallest file sizes while maintaining watchable quality |
| **Streaming** | Optimized for streaming-friendly formats |
{.dense}

Custom quality profiles are currently a work in progress.

## 2.3 Subtitles

Cinephage supports 8 subtitle providers with 80+ languages. Subtitles can be configured to auto-search on import with language profile preferences. Navigate to **Settings → Subtitles** to configure your preferred providers and languages.

## 2.4 Monitoring

Cinephage can automatically search for missing content, quality upgrades, new episodes, and items not meeting your quality cutoff. Configure monitoring schedules under **Settings → Monitoring**.

> 
> Monitoring is currently marked as **experimental** and may have bugs. Keep an eye on logs if you enable automated tasks.
{.is-info}



# <img src="/patreon-light.png" class="tab-icon"> 4 · Video
[![](/2026-02-25-cinephage-the-all-in-one-app-promo-card.png)](https://www.patreon.com/posts/cinephage-all-in-150415696)