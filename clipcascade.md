---
title: ClipCascade
description: A guide to deploying ClipCascade
published: true
date: '2026-11-04T16:23:57.825Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:03:43.844Z'
---

# What is ClipCascade?

ClipCascade is a lightweight, open-source utility that automatically syncs your clipboard across multiple devices—no manual input required. It ensures seamless sharing with robust end-to-end encryption, providing a secure and reliable clipboard experience across workstations.

# :logos:docker-icon: 1 · Deploy ClipCascade
```yaml
services:
  clipcascade:
    image: sathvikrao/clipcascade:latest
    ports:
      - "8080:8080"  
    restart: unless-stopped
    volumes:
      - ./cc_users:/database
    environment:
      - CC_MAX_MESSAGE_SIZE_IN_MiB=1
      - CC_P2P_ENABLED=false
      # - CC_ALLOWED_ORIGINS=https://clipcascade.example.com
      # - CC_SIGNUP_ENABLED=false
```

# 2 · Logging In	
1. Navigate to `http://IP:8080`
1. Username is `admin` and password is `admin123`

# 3 · Client Software
Visit the [Releases Page](https://github.com/Sathvik-Rao/ClipCascade/releases) on GitHub to see the latest versions of the client software.


