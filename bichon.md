---
title: Bichon
description: A guide to deploying Bichon
published: true
date: 2026-01-15T15:28:17.084Z
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
    image: rustmailer/bichon:latest

```