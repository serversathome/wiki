---
title: Sonarr Analyzer
description: A guide to deploying Sonarr Analyzer
published: true
date: '2026-11-04T23:31:40.831Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:08:38.032Z'
---

# 📊 What is Sonarr Analyzer?
A web application built with Streamlit for analyzing and monitoring your Sonarr library. It tracks average file size per episode, flags outliers, keeps analysis history, forecasts storage growth, and can run scheduled analyses. It supports multiple users and multiple Sonarr instances.

# :logos:docker-icon: 1 · Deploy Sonarr Analyzer
```yaml
services:
  sonarr-analyzer:
    image: martitoci/sonarr-analyzer:latest
    container_name: sonarr-analyzer
    ports:
      - "8501:8501"
    volumes:
      - /mnt/tank/configs/sonarranalyzer:/app/data
    restart: unless-stopped
```

The container runs as UID **1000** (`appuser`), so give the config folder to that user before deploying:

```bash
chown -R 1000:1000 /mnt/tank/configs/sonarranalyzer
```

# 2 · First Run

::block-steps
1. Navigate to `http://IP:8501`
1. On first launch you'll see the **Create Admin Account** screen. Pick a username (min 3 characters) and password (min 8 characters)
1. Log in with the new admin account
1. Go to **Configuration** in the sidebar and add your Sonarr instance (name, **Sonarr URL**, **API Key**), then click **Save Instance**
1. Activate the instance and run your first analysis
::

