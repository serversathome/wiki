---
title: Arr Queue Cleaner
description: A guide to deploy Arr Queue Cleaner
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:11.738Z
---

# What is Arr Queue Cleaner?
Automated queue cleaner for Sonarr that removes stuck downloads based on configurable rules.

<div class="glance">
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/thelegendtubaguy/arrqueuecleaner"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Arr Queue Cleaner

```yaml
services:
  arr-queue-cleaner:
    image: ghcr.io/thelegendtubaguy/arrqueuecleaner:latest
    environment:
      - SONARR_HOST=http://sonarr:8989
      - SONARR_API_KEY=your_api_key_here
      - REMOVE_QUALITY_BLOCKED=true
      - BLOCK_REMOVED_QUALITY_RELEASES=false
      - REMOVE_ARCHIVE_BLOCKED=true
      - BLOCK_REMOVED_ARCHIVE_RELEASES=false
      - REMOVE_EXECUTABLE_BLOCKED=true
      - REMOVE_NO_FILES_RELEASES=true
      - BLOCK_REMOVED_NO_FILES_RELEASES=true
      - REMOVE_NOT_AN_UPGRADE=true
      - REMOVE_SERIES_ID_MISMATCH=true
      - BLOCK_REMOVED_SERIES_ID_MISMATCH_RELEASES=false
      - REMOVE_EPISODE_COUNT_MISMATCH=false
      - BLOCK_REMOVED_EPISODE_COUNT_MISMATCH_RELEASES=false
      - REMOVE_UNDETERMINED_SAMPLE=false
      - BLOCK_REMOVED_UNDETERMINED_SAMPLE=false
      - REMOVE_POTENTIALLY_DANGEROUS_FILES=true
      - BLOCK_POTENTIALLY_DANGEROUS_FILES=true
      - REFRESH_TBA_TITLE_SERIES=false
      - DRY_RUN=false
      - SCHEDULE=*/5 * * * *
      - LOG_LEVEL=info
    restart: unless-stopped
```

The rules set to `true` above are the ones upstream recommends. Set any of them to `false` to turn that rule off.

> Have more than one Sonarr server? Replace `SONARR_HOST` and `SONARR_API_KEY` with a `SONARR_INSTANCES` JSON list (or a file via `SONARR_INSTANCES_FILE`). See the [official docs](https://github.com/thelegendtubaguy/ArrQueueCleaner#multiple-sonarr-instances) for the format.
{.is-info}

> See [this official docs](https://github.com/thelegendtubaguy/ArrQueueCleaner#environment-variables) for a list of environment variable explanations
{.is-info}
