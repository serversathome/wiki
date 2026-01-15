---
title: Sommelierr
description: A guide to deploying Sommelierr
published: true
date: 2026-01-15T15:31:49.977Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:08:35.736Z
---

# What is Sommelierr?
This container generates random recommendation from your Radarr and Sonarr libraries.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Sommelierr

```yaml
services:
  sommelierr:
    image: ghcr.io/rare-magma/sommelierr:latest
    read_only: true
    pull_policy: always
    cap_drop:
      - ALL
    security_opt:
      - no-new-privileges:true
    env_file:
      - .env
    ports:
      - 8083:8080
```

## env File

```yaml
EXCLUDE_LABEL=watched
RADARR_HOST=http://
RADARR_API_KEY=
SONARR_HOST=http://
SONARR_API_KEY=
PORT=8080

```