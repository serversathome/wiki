---
title: Sonarr Analyzer
description: A guide to deploying Sonarr Analyzer
published: true
date: 2026-01-15T15:31:51.520Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:08:38.032Z
---

# 📊 What is Sonarr Analyzer?
A web application built with Streamlit for analyzing average file size per episode across TV series managed by Sonarr.
# <img src="/docker.png" class="tab-icon"> 1 · Deploy Sonarr Analyzer
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