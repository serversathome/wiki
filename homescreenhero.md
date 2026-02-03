---
title: Homescreen Hero
description: A guide to deploying Homescreen Hero
published: true
date: 2026-02-03T12:23:06.048Z
tags: 
editor: markdown
dateCreated: 2026-02-03T12:22:46.645Z
---

# <img src="/homescreen-hero.png" class="tab-icon"> What is HomeScreen Hero?

**HomeScreen Hero** is a self-hosted web app that keeps your Plex home screen fresh by automatically rotating collections on a schedule or manually via a modern FastAPI + React dashboard. Beyond collection rotation, it provides server insights, useful Plex management tools, and integrations with popular services like Trakt, Letterboxd, MDBList, Tautulli, and Overseerr.

**Key Features:**
- Web Dashboard for managing your Plex Server and curating homescreens
- Automated Collection Rotation with multiple strategies (random, weighted, LRU)
- First-Time Setup Wizard for easy configuration
- 3rd Party List Integrations (Trakt, Letterboxd, MDBLists)
- Widgets for Tautulli and Overseerr
- Tools for managing Date Added timestamps, Watch History, and generating Unwatched Reports

# <img src="/docker.png" class="tab-icon"> 1 · Deploy HomeScreen Hero

```yaml
services:
  homescreen-hero:
    image: trentferguson/homescreen-hero:latest
    container_name: homescreen-hero
    environment:
      - HSH_PORT=8000
      - HSH_PLEX_URL=http://YOUR_PLEX_IP:32400
      - HSH_PLEX_TOKEN=your_plex_token
      # Optional: Enable authentication
      # - HSH_AUTH_PASSWORD=your_password
      # - HSH_AUTH_SECRET_KEY=your_secret_key
      # Optional: Integrations
      # - HSH_TRAKT_CLIENT_ID=your_trakt_client_id
      # - HSH_MDBLIST_API_KEY=your_mdblist_api_key
      # - HSH_TAUTULLI_API_KEY=your_tautulli_api_key
      # - HSH_TAUTULLI_BASE_URL=http://localhost:8181
      # - HSH_SEERR_API_KEY=your_seerr_api_key
      # - HSH_SEERR_BASE_URL=http://localhost:5055
    restart: unless-stopped
    ports:
      - "8000:8000"
    volumes:
      - /mnt/tank/configs/homescreenhero:/data
```

1. Replace `YOUR_PLEX_IP` with your Plex server IP address
2. Replace `your_plex_token` with your [Plex authentication token](https://support.plex.tv/articles/204059436-finding-an-authentication-token-x-plex-token/)
3. Deploy the stack and navigate to `http://your-server-ip:8000`
4. Complete the **Setup Wizard** to configure your libraries and rotation settings

> 
> The Setup Wizard will guide you through the initial configuration. You don't need to edit config files manually unless you want to.
{.is-info}

# 2 · Configuration

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `HSH_PORT` | Port for the web UI (default: 8000) | Yes |
| `HSH_PLEX_URL` | Your Plex server URL | Yes |
| `HSH_PLEX_TOKEN` | Your Plex authentication token | Yes |
| `HSH_AUTH_PASSWORD` | Password for web UI login | If auth enabled |
| `HSH_AUTH_SECRET_KEY` | JWT secret for authentication | If auth enabled |
| `HSH_TRAKT_CLIENT_ID` | Trakt API client ID | If Trakt enabled |
| `HSH_MDBLIST_API_KEY` | MDBList API key | If MDBList enabled |
| `HSH_TAUTULLI_API_KEY` | Tautulli API key | If Tautulli enabled |
| `HSH_TAUTULLI_BASE_URL` | Tautulli URL (default: http://localhost:8181) | No |
| `HSH_SEERR_API_KEY` | Overseerr/Jellyseerr API key | If Seerr enabled |
| `HSH_SEERR_BASE_URL` | Overseerr URL (default: http://localhost:5055) | No |
{.dense}

## Rotation Strategies

HomeScreen Hero supports three rotation strategies for cycling through your collections:

| Strategy | Description |
|----------|-------------|
| `random` | Groups processed in config order, collections selected randomly |
| `weighted` | Groups processed by weight (highest first), collections selected randomly |
| `lru` | Least recently used collections selected first for fair rotation |
{.dense}

All strategies respect the `min_gap_rotations` setting to prevent collections from appearing too frequently.

## Configuration File

Advanced settings can be configured via `config.yaml` stored in the `/data` volume. Key sections include:
- **plex** – Server URL, token, and libraries to manage
- **rotation** – Interval, max collections, and rotation strategy
- **groups** – Named pools of collections with min/max picks and weights
- **trakt/mdblist** – Third-party list sync configuration

# 3 · Built-in Tools

HomeScreen Hero includes several useful tools for Plex server management:

## Date Added Editor
Fix the "Date Added" timestamp on movies and shows that were redownloaded to your library. Options include setting a custom date, 30 days ago, or matching the media's original release date.

## Watch History Cleaner
Mark TV shows as unwatched to fix issues with Plex's "Continue Watching" row. Useful when shows disappear from Continue Watching or you want to start a fresh rewatch.

## Unwatched Report
Find content that's collecting dust in your library by searching for items that have never been watched or haven't been watched within a specified time period. Requires Tautulli integration. Results can be exported to CSV for library cleanup decisions.

# 4 · Integrations

## Trakt
Sync collections from your Trakt lists. Get your Client ID from [Trakt API Applications](https://trakt.tv/oauth/applications).

## MDBList
Create collections from curated MDBList lists. Get your API key from [MDBList Preferences](https://mdblist.com/preferences/).

## Letterboxd
Import collections from public Letterboxd lists.

## Tautulli Widget
Add a Tautulli widget to your dashboard for server activity insights. Find your API key in Tautulli Settings → Web Interface → API.

## Overseerr Widget
Add an Overseerr widget to your dashboard for request management. Find your API key in Overseerr Settings → General.

# 5 · Resources

- **GitHub:** https://github.com/trentferguson/homescreen-hero
- **Live Demo:** https://demo.homescreenhero.com
- **Discord:** https://discord.gg/yQ8pJzURsr

# <img src="/youtube.png" class="tab-icon"> 6 · Video

*No video yet — check back soon!*