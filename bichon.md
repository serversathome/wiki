---
title: Bichon
description: A guide to deploying Bichon
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:27.886Z
---

# <img src="/bichon.png" class="tab-icon"> What is Bichon?
Bichon is an open-source email archiving system that synchronizes emails from IMAP servers, indexes them for full-text search, and provides a REST API for programmatic access. Unlike email clients, Bichon is designed for archiving and searching rather than sending/receiving emails. It runs as a standalone server application that continuously synchronizes configured email accounts and maintains a searchable local archive. Built in Rust, it requires no external dependencies and provides fast, efficient email archiving, management, and search through a built-in WebUI. 

<div class="glance">
  <div><span>Port</span><b><code>15630</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/rustmailer/bichon"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Bichon
```yaml
services:
  bichon:
    container_name: bichon
    ports:
      - 15630:15630
    volumes:
      - /mnt/tank/configs/bichon:/data
    environment:
      - BICHON_LOG_LEVEL=info
      - BICHON_ROOT_DIR=/data
      - BICHON_ENCRYPT_PASSWORD=CHANGEME
    image: rustmailer/bichon:latest
    restart: unless-stopped
```

1. Generate an encryption password with `openssl rand -hex 32` and paste it in place of `CHANGEME`
2. Deploy the stack

> `BICHON_ENCRYPT_PASSWORD` encrypts the IMAP passwords and OAuth tokens Bichon stores. Save it somewhere safe. If you lose or change it, Bichon can no longer read your saved account credentials.
{.is-warning}

# 2 · Logging In
1. Browse to `http://your-server-ip:15630`
2. Log in with username `admin` and password `admin@bichon`
3. Change the password right away under **Settings → Profile**
