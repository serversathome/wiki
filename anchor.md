---
title: Anchor
description: A guide to deploying Anchor
published: true
date: 2026-02-04T13:21:46.457Z
tags: 
editor: markdown
dateCreated: 2026-02-04T13:21:46.457Z
---

# <img src="/anchor-notes.png" class="tab-icon"> What is Anchor?

**Anchor** is an offline-first, self-hostable note-taking application designed for speed, privacy, simplicity, and reliability. Notes are stored locally, fully editable offline, and automatically sync across devices when online. Anchor offers both web and mobile (Android) interfaces, making it a great alternative to cloud-dependent note apps.

## Features

- Rich text editor with formatting (bold, italic, underline, headings, lists, checkboxes)
- Note sharing with other users (viewer or editor permissions)
- Tags system with custom colors for organization
- Customizable note backgrounds (solid colors and patterns)
- Pin and archive notes for quick access
- Full-text search by title or content
- Trash with recovery period
- Offline-first with automatic sync when online
- Admin panel for user management and registration control

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Anchor

```yaml
services:
  anchor:
    image: ghcr.io/zhfahim/anchor:latest
    container_name: anchor
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - /mnt/tank/configs/anchor:/data
```


# 2 · Configuration

## 2.1 Environment Variables

Most users can run Anchor with defaults. For advanced configuration, these environment variables are available:

| Variable | Default | Description |
|----------|---------|-------------|
| `JWT_SECRET` | (auto-generated) | Auth token secret (persisted in `/data`) |
| `PG_HOST` | (empty) | External Postgres host (leave empty for embedded) |
| `PG_PORT` | `5432` | Postgres port |
| `PG_USER` | `anchor` | Postgres username |
| `PG_PASSWORD` | `password` | Postgres password |
| `PG_DATABASE` | `anchor` | Database name |
| `USER_SIGNUP` | (not set) | Sign up mode: `disabled`, `enabled`, or `review` |
{.dense}


# 3 · Mobile App

Anchor provides an Android mobile app that syncs with your self-hosted server.

1. Visit the [GitHub Releases](https://github.com/zhfahim/anchor/releases) page
2. Download the latest APK:
   - **Universal APK** (`anchor-{version}.apk`) - Recommended for most users
   - **Architecture-specific APKs** - Smaller file sizes for specific devices
3. Install and connect to your Anchor server URL

> 
> The mobile app works offline and syncs changes when connected to your server.
{.is-success}


# <img src="/youtube.png" class="tab-icon"> 4 · Video

*Video coming soon*