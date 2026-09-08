---
title: Rackula
description: A guide to deploying Rackula
published: true
date: 2026-01-15T15:31:14.756Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:51.532Z
---

# <img src="/rackula.png" class="tab-icon"> What is Rackula?
Plan your rack layout. Drag your devices in, move them around, export it. It runs in your browser. You can close the tab whenever you want.

<div class="glance">
  <div><span>Port</span><b><code>8080</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/rackulalives/rackula"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Rackula
```yaml
services:
  Rackula:
    image: ghcr.io/rackulalives/rackula:latest
    ports:
      - "8080:80"
    restart: unless-stopped
```