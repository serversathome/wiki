---
title: Arr Queue Cleaner
description: A guide to deploy Arr Queue Cleaner
published: true
date: 2026-01-15T15:28:04.792Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:11.738Z
---

# What is Arr Queue Cleaner?
Automated queue cleaner for Sonarr that removes stuck downloads based on configurable rules.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Arr Queue Cleaner

```yaml
services:
  arr-queue-cleaner:
    image: ghcr.io/thelegendtubaguy/arrqueuecleaner:latest
    environment:
      - SONARR_HOST=http://sonarr:8989
      - SONARR_API_KEY=your_api_key_here
      - REMOVE_QUALITY_BLOCKED=false
      - BLOCK_REMOVED_QUALITY_RELEASES=false
      - REMOVE_ARCHIVE_BLOCKED=false
      - BLOCK_REMOVED_ARCHIVE_RELEASES=false
      - REMOVE_NO_FILES_RELEASES=false
      - BLOCK_REMOVED_NO_FILES_RELEASES=false
      - REMOVE_SERIES_ID_MISMATCH=false
      - BLOCK_REMOVED_SERIES_ID_MISMATCH_RELEASES=false
      - REMOVE_UNDETERMINED_SAMPLE=false
      - BLOCK_REMOVED_UNDETERMIND_SAMPLE=false
      - DRY_RUN=false
      - SCHEDULE=*/5 * * * *
      - LOG_LEVEL=info
    restart: unless-stopped
```

> See [this official docs](https://github.com/thelegendtubaguy/ArrQueueCleaner#environment-variables) for a list of environment variable explanations
{.is-info}
