---
title: IronCalc
description: A guide to deploying IronCalc
published: true
date: '2026-11-04T16:23:39.264Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:05:33.456Z'
---

# What is IronCalc?

IronCalc is a new, modern, work-in-progress spreadsheet engine and set of tools to work with spreadsheets in diverse settings.

# :logos:docker-icon: 1 · Deploy IronCalc
IronCalc now publishes prebuilt images, so you no longer need to clone the repo. Create a new stack named `ironcalc` in Dockge with this compose:

```yaml
services:
  server:
    image: ghcr.io/ironcalc/ironcalc-server:0.9.1
    restart: unless-stopped

  caddy:
    image: ghcr.io/ironcalc/ironcalc-caddy:0.9.1
    ports:
      - 2080:2080
    depends_on:
      - server
    restart: unless-stopped
```

Then open `http://IP:2080` in your browser.

