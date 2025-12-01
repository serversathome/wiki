---
title: Bichon
description: A guide to deploying Bichon
published: true
date: 2025-12-01T22:18:36.642Z
tags: 
editor: markdown
dateCreated: 2025-12-01T22:18:36.642Z
---

# <img src="/bichon.png" class="tab-icon"> What is Bichon?
Bichon is an open-source email archiving system that synchronizes emails from IMAP servers, indexes them for full-text search, and provides a REST API for programmatic access. Unlike email clients, Bichon is designed for archiving and searching rather than sending/receiving emails. It runs as a standalone server application that continuously synchronizes configured email accounts and maintains a searchable local archive. Built in Rust, it requires no external dependencies and provides fast, efficient email archiving, management, and search through a built-in WebUI. 
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