---
title: Sonarr Analyzer
description: A guide to deploying Sonarr Analyzer
published: true
date: 2025-11-06T18:56:31.013Z
tags: 
editor: markdown
dateCreated: 2025-11-06T18:56:31.013Z
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