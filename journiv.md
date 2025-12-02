---
title: Journiv
description: A guide to deploy Journiv
published: true
date: 2025-12-02T01:21:26.115Z
tags: 
editor: markdown
dateCreated: 2025-12-02T01:21:26.115Z
---

# What is Journiv?
Journiv is a self-hosted private journal. It features comprehensive journaling capabilities including mood tracking, prompt-based journaling, media uploads, analytics, and advanced search with a clean and minimal UI.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Journiv
```yaml
services:
  journiv-app:
    container_name: journiv
    ports:
      - 8000:8000
    environment:
      - SECRET_KEY=your-secret-key-here
      - DOMAIN_NAME=10.99.0.242
    volumes:
      - /mnt/tank/configs/journiv:/data
    restart: unless-stopped
    image: swalabtech/journiv-app:latest
```