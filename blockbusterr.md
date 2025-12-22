---
title: Blockbusterr
description: A guide to deploying Blockbusterr
published: true
date: 2025-12-22T13:43:10.132Z
tags: 
editor: markdown
dateCreated: 2025-12-22T13:43:10.132Z
---

# What is Blockbusterr?

Automatically add trending, popular, and highly-rated movies and TV shows from Trakt.tv to your Radarr and Sonarr instances.

- **15 Automated Jobs**: Sync movies and TV shows from various Trakt lists
	- Trending, Popular, Box Office
  - Favorited, Played, Watched, Collected (by time period)
  - Anticipated content
- **Content Filtering**: Fine-grained control over what gets added to your library
	- Filter by country, language, genre, keywords
  - Block specific networks (Hallmark, Nickelodeon, etc.)
  - Set runtime and year ranges
  - Blacklist specific TMDB/TVDB IDs
- **Dual Integration Modes**:
	- **Direct Mode**: Add content directly to Radarr/Sonarr
  - **Jellyseerr Mode**: Request content via Jellyseerr/Overseerr with approval workflows
- **Web UI**: Manage jobs, filters, view activity logs, and configure settings
- **Flexible Scheduling**: Use simple durations (1h, 30m) or cron expressions (0 */2 * * *)
- **Smart Duplicate Detection**: Prevents adding content that's already requested/added
- **Activity Logging**: Track all job executions and API actions
- **Configurable Limits**: Control how many items to add per job
- **Time Periods**: Choose weekly, monthly, yearly, or all-time for historical lists


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Blockbusterr
```yaml
services:
  blockbusterr:
    image: ghcr.io/mahcks/blockbusterr:latest
    container_name: blockbusterr
    restart: unless-stopped
    ports:
      - 9090:9090
    volumes:
      - /mnt/tank/configs/blockbusterr:/app/data
      
    environment:
      # Optional: Specify custom config file location
      # CONFIG_PATH: /app/config
      
      # Optional: Override config values with environment variables
      # TRAKT_CLIENT_ID: your_client_id
      # TRAKT_CLIENT_SECRET: your_client_secret
      # RADARR_URL: http://radarr:7878
      # RADARR_API_KEY: your_radarr_key
      # SONARR_URL: http://sonarr:8989
      # SONARR_API_KEY: your_sonarr_key
      # DISABLE_UI: true  # Set to disable Web UI (enabled by default)
      - TZ=America/New_York  # Set your timezone
```