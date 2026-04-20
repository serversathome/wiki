---
title: Newtarr
description: A guide to deploying Newtarr
published: true
date: 2026-04-20T13:08:40.526Z
tags: 
editor: markdown
dateCreated: 2026-03-14T00:05:42.713Z
---

# What is NewtArr?

**NewtArr** is a streamlined \*arr media library hunter — a community fork of Huntarr v6.6.3, maintained by [ElfHosted](https://store.elfhosted.com). It continuously scans your Sonarr, Radarr, Lidarr, Readarr, and Whisparr libraries for missing content and items that need quality upgrades, then automatically triggers searches while being gentle on your indexers.

The original Huntarr project was abandoned after the developer introduced telemetry, obfuscated code, and potential security concerns. NewtArr is based on the last clean release (v6.6.3) before those controversial changes, with all telemetry and update-check code stripped out.


# <img src="/docker.png" class="tab-icon"> 1 · Deploy NewtArr

```yaml
services:
  newtarr:
    image: ghcr.io/elfhosted/newtarr:v1.0.0
    container_name: newtarr
    restart: unless-stopped
    ports:
      - "9705:9705"
    volumes:
      - /mnt/tank/configs/newtarr:/config
    environment:
      - TZ=America/New_York
```


# 2 · Configuration

## 2.1 Connect Your *Arr Apps

All configuration is done through the web UI. For each *arr app you want NewtArr to monitor:

1. Navigate to the app's section in the NewtArr UI
2. Enter the **URL** of your *arr instance (e.g. `http://sonarr:8989`)
3. Enter the **API Key** — you can find this in each *arr app under **Settings → General**
4. Click **Save**


> NewtArr supports multiple instances of the same app. If you run more than one Sonarr or Radarr instance, you can add them all.
{.is-info}

## 2.2 Search Settings

Configure how aggressively NewtArr hunts for your missing media:

| Setting | Description |
|---------|-------------|
| **Items per cycle** | How many missing items to search for in each pass |
| **Sleep duration** | Time between search cycles |
| **API rate limits** | Hourly cap on API calls to protect your indexers |


> 
> Start conservative with search settings. If your indexers start rate-limiting you, increase the sleep duration and reduce items per cycle.
{.is-info}


# 3 · Security Notes

A comprehensive security audit of the v6.6.3 codebase has been performed and is available in the [SECURITY-AUDIT.md](https://github.com/elfhosted/newtarr/blob/main/SECURITY-AUDIT.md) on GitHub.

Key takeaways:
- **No telemetry, phone-home code, or obfuscated code** was found in the v6.6.3 codebase
- The original codebase has some security considerations (hardcoded secret key, weak password hashing) which are mitigated when running behind an SSO proxy
- Standalone users should review the audit and apply recommended mitigations

> 
> If you are running NewtArr without a reverse proxy or SSO, review the [security audit](https://github.com/elfhosted/newtarr/blob/main/SECURITY-AUDIT.md) for recommended hardening steps.
{.is-danger}

# 4 · Why NewtArr Over Huntarr?

The original Huntarr project went through massive scope creep after v6.6.3:

- **v7.x** added a full request system (Requestarr), Prowlarr integration, Plex OAuth, and a massive database layer
- **v9.x** became a completely different application with built-in Usenet and BitTorrent download clients, effectively trying to replace your entire *arr stack

NewtArr sticks to the original purpose: **hunt missing episodes, movies, and media** — nothing more, nothing less. It supports multi-instance configurations and Swaparr (stalled download handling) without any of the bloat.

> 
> NewtArr is licensed under GPL-3.0. It is a community-maintained fork — check the [GitHub repo](https://github.com/elfhosted/newtarr) for the latest updates and to report issues.
{.is-info}

# <img src="/youtube.png" class="tab-icon"> 5 · Video

[![](/2026-03-24-replacing-huntarr-with-newtarr--promo-card.png)](
https://www.patreon.com/posts/replacing-with-153191561)