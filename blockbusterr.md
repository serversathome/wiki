---
title: Blockbusterr
description: A guide to deploying Blockbusterr
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:30.104Z
---

# What is Blockbusterr?

Automated media discovery with rule-based decision making. Blockbusterr follows trends, lists and watchlists from TMDB, Simkl, Trakt, MDBList and Letterboxd, checks every movie or show against your rules, ranks the candidates and sends the winners to Radarr, Sonarr, Jellyseerr or Seerr.

- **Preview-first jobs**: Inspect candidates and rule decisions before turning on delivery
- **Reusable rules**: Assign one movie or show policy to many jobs, or make a job-specific copy
- **Lists and watchlists**: Follow Trakt, TMDB, MDBList and (experimental) public Letterboxd lists. Trakt is optional
- **Recipes and custom jobs**: Start from a built-in recipe or configure the whole discovery flow yourself
- **Ranked selection**: Let jobs compete for a shared number of movie or show slots
- **Delivery safeguards**: Delivery budgets, repeat handling and title exceptions keep your library from turning into a firehose
- **Two delivery paths**: Add directly to Radarr/Sonarr, or request through Jellyseerr/Seerr
- **Decision history**: Every title gets an outcome, reason, score and source, so you can see why it was added or skipped
- **Flexible Scheduling**: Use simple durations (1h, 30m) or cron expressions (0 */2 * * *)

<div class="glance">
  <div><span>Port</span><b><code>9090</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/mahcks/blockbusterr"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>


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
      - TZ=America/New_York  # Set your timezone
      # - BLOCKBUSTERR_AUTH_TOKEN=CHANGEME  # Protects the UI and API, at least 32 random characters
```

> Without `BLOCKBUSTERR_AUTH_TOKEN` the web UI and API are open to anyone on your network. To lock them down, generate a token with `openssl rand -hex 32`, uncomment the line and paste it in place of `CHANGEME`.
{.is-info}

> **Upgrading from v1?** `latest` is now v2, a major rewrite. Stop the container and back up `/mnt/tank/configs/blockbusterr` first. For the first v2 start, add `BLOCKBUSTERR_DRY_RUN=true` so scheduled jobs only preview instead of adding content, and remove it once the previews look right. v2 runs as UID/GID `10001:10001` and fixes ownership of the data folder itself. Follow the [upgrade guide](https://blockbusterr.dev/v2/getting-started/upgrading-to-v2/).
{.is-warning}
# <img src="/patreon-light.png" class="tab-icon"> 2 · Video

[![](/2026-01-01-auto-add-content-with-blockbuste-promo-card.png)](https://www.patreon.com/posts/auto-add-content-147151544)