---
title: Agregarr
description: A guide to deploying Agregarr
published: true
date: 2025-09-01T12:22:28.908Z
tags: 
editor: markdown
dateCreated: 2025-09-01T12:22:28.908Z
---

# ![](/agregarr.png){class="tab-icon"} What is Agregarr?
Agregarr keeps your Plex Home and Recommended fresh by frequently updating it with Collections from various sources, including Trakt, IMDb, TMdb and Letterboxd, as well as generated Collections from Tautulli Statistics, and Overseerr Requests. It has various options for downloading missing media, including as requests through Overseerr. Collections can be reordered on the Home/Recommended and Library tabs independently, and can have time periods or days set for their visibility in Plex.


- Public Lists: Add public lists from Trakt, IMDb, TMDB and Letterboxd, with presets and custom list options
- Overseerr Requests: Generate Collections either for each users requests (only visible to that user), or for All Requests
- Tautulli Statistics: Generate Collections based on the Most Popular content on your server
- Independent Reordering: Control the order in which Collections appear across the Home/Recommended screens and the Library tab independetly
- Keeps Plex Updated: Collections will be be updated on every sync (default 12 hours)
- Template System: Easily set collection names with flexible templating
- Time Restrictions: Schedule collections to be active only during specific time periods
- Library Management: Organize collections across multiple Plex libraries
- Exising Collection Integration: Any pre-existing Collections in Plex and Default Hubs (Recently Added etc) can be managed alongside Agregarr Collections
- Collection Statistics: Dashboard showing Most Popular Collections (from Tautulli), and recently added Missing Items


# <img src="/docker.png" class="tab-icon"> 1 · Deploy Agregarr

```yaml
services:
  agregarr:
    image: agregarr/agregarr:latest
    container_name: agregarr
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/agregarr:/app/config
    ports:
      - 7171:7171
```