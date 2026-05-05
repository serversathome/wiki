---
title: Capacitarr
description: A guide to deploying Capacitarr
published: true
date: 2026-05-05T19:29:32.558Z
tags: 
editor: markdown
dateCreated: 2026-03-21T11:32:55.609Z
---

# What is Capacitarr?

**Capacitarr** is an intelligent media library capacity manager for the \*arr ecosystem. It scores every media item across six dimensions — watch history, recency, file size, ratings, age, and series status — then removes the least-valuable items first when disk space runs low. A visual rule builder lets you protect specific content from ever being deleted.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Capacitarr

```yaml
services:
  capacitarr:
    image: ghcr.io/ghent/capacitarr:stable
    container_name: capacitarr
    environment:
      - PUID=568
      - PGID=568
      - JWT_SECRET=change-me-to-a-random-string
    restart: unless-stopped
    ports:
      - "2187:2187"
    volumes:
      - /mnt/tank/configs/capacitarr:/config
```

 
> Do **not** leave the `JWT_SECRET` as the default value. Generate a random string (e.g. `openssl rand -hex 32`) and use that instead.
{.is-danger}


# 2 · Configuration

## 2.1 Connect Your Stack

After first launch, you'll need to point Capacitarr at your existing services. For each integration, you provide the **URL** and **API key** of the service. Capacitarr supports 9 integrations:

| Service | Type |
|---|---|
| Sonarr | \*arr app (TV) |
| Radarr | \*arr app (Movies) |
| Lidarr | \*arr app (Music) |
| Readarr | \*arr app (Books) |
| Plex | Media server |
| Jellyfin | Media server |
| Emby | Media server |
| Tautulli | Plex analytics |
| Overseerr | Request manager |


## 2.2 Scoring & Rules

Capacitarr scores every item in your library across six weighted dimensions:

| Dimension | Description |
|---|---|
| Watch History | How recently and how often the item has been watched |
| Recency | How recently the item was added to your library |
| File Size | How much disk space the item consumes |
| Ratings | Community and critic ratings |
| Age | How old the media itself is (release date) |
| Series Status | Whether a series is still airing, ended, or canceled |


You can configure:

- **Scoring Weights** — adjust how much each of the six dimensions matters to your cleanup decisions
- **Visual Rule Builder** — create rules using four priority levels: `always_keep`, `prefer_keep`, `prefer_delete`, and `always_delete`
- **Protected Content** — tag specific items so certain media is never deleted

Every score is transparent and fully explainable through the score detail view.

## 2.3 Deletion Workflow

Capacitarr uses a safety-first approach to media cleanup:

- **Dry-run mode** — preview what would be removed without taking action
- **Approval queue** — review and approve deletions before they happen
- **Safety guards** — built-in protections prevent accidental mass deletion
- **Audit log** — a complete record of every decision and action taken

> 
> Capacitarr will **never** delete media without your approval unless you explicitly configure it to do so. The default mode requires manual review of all deletion candidates.
{.is-success}

## 2.4 Real-Time Updates

The dashboard uses Server-Sent Events (SSE) to push engine state, deletions, and activity to the browser instantly — no polling required. There are 40 typed event types to keep you informed of everything happening in your library.

# <img src="/patreon-light.png" class="tab-icon"> 3 · Video

[![](/2026-04-07-capacitarr--let-your-arr-stack-promo-card.png)](https://www.patreon.com/posts/capacitarr-let-153757406)