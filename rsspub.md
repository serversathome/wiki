---
title: RSSPub
description: A guide to deploying RSSPub
published: true
date: 2025-12-29T14:58:56.063Z
tags: 
editor: markdown
dateCreated: 2025-12-29T14:58:56.063Z
---

# What is RSSPub?

rsspub is a self-hosted Rust application that turns your favorite RSS/Atom feeds/ Read Later articles into a personal daily newspaper (EPUB). It fetches articles, processes images, and bundles everything into an EPUB file that you can read on your e-reader or tablet.

It also serves an OPDS feed, making it easy to download the generated EPUBs directly to compatible readers (like KOReader, Moon+ Reader, etc.).

# <img src="/docker.png" class="tab-icon"> 1 · Deploy RSSPub
```yaml
services:
  rsspub:
    ports:
      - 3000:3000
    image: harshit181/rsspub:latest
    volumes:
      - /mnt/tank/configs/rsspub:/app/db
    environment:
      - RUST_LOG=info,html5ever=error
      - RPUB_USERNAME=admin
      - RPUB_PASSWORD=admin
    restart: unless-stopped
    container_name: rsspub
```