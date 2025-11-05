---
title: Overview Map
description: Overview map of how all the *arr components of a media server fit together
published: true
date: 2025-11-05T20:50:12.733Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:30:38.032Z
---

# Visual Map
![arr_stack_2026.png](/arr_stack_2026.png)
A full \*arr suite is composed of many apps that all talk to each other to automate your life. This is what they are and what they do:

-   **Prowlarr** - this keeps track of indexers (sites that search for content) and manages their settings to pass them along to other apps
- **Profilarr** - Sync quality profiles to Radarr/Sonarr
-   **Radarr** - app for managing movies
-   **Sonarr** - app for managing tv shows
-   **Unpackerr** - while not an official arr, still super useful for when you get .rar content and need it to be unzipped/unpacked
-   **qBittorrent** - download client for torrents
-   **Emby**/**Plex**/**Jellyfin** - media servers so u can watch your stuff on multiple devices
-   **Jellyseerr** - a single interface for both radarr/sonarr which also does recommendations
-   **AirVPN** - VPN to protect your identity while torrenting
-   **Bazarr** - provides subtitles in case they are missing
- **Huntarr** - finds missing and upgrades low quality media
- **Wizarr** - creates invites to media server for users
