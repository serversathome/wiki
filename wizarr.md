---
title: Wizarr
description: A guide to deploying Wizarr
published: true
date: 2025-06-13T03:14:22.611Z
tags: 
editor: markdown
dateCreated: 2025-04-11T10:21:57.578Z
---

![wizarr.png](/wizarr.png)

# What is Wizarr?
Wizarr is an automated user invitation system compatible with Plex, Jellyfin and Emby. You can create a unique link, share it with a user, and they will be invited to your Media Server after they complete the simple signup process!

# Installation
```yaml
services:
  wizarr:
    container_name: wizarr
    restart: unless-stopped
    image: ghcr.io/wizarrrr/wizarr:latest
    ports:
      - 5690:5690
    environment:
      - APP_URL=https://example.domain.com
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/wizarr:/data/database
      - /mnt/tank/configs/wizarr/wizard:/data/wizard_steps
```
1. Change the `APP_URL` to match your domain
1. The config directory assumes you follow the standard Folder Structure. If you do not it needs to be modified to the correct volume.

> Use **Generic** for the dataset preset in TrueNAS
{.is-info}


# Wizarr Configuration

Wizarr has moved their docs. 🙁

> You will first need to create an API Key for Jellyfin by navigating to the **Administration Dashboard** → **API Keys** and clicking the `+` in the top left
{.is-info}

# Video

![2025-04-14-wizarr-share-your-plex-jellyfi-promo-card.png](/2025-04-14-wizarr-share-your-plex-jellyfi-promo-card.png)
[Watch it on Patreon!](https://www.patreon.com/posts/wizarr-share-or-126460048?utm_medium=clipboard_copy&utm_source=copyLink&utm_campaign=postshare_creator&utm_content=join_link)
