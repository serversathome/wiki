---
title: IronCalc
description: A guide to deploying IronCalc
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:05:33.456Z
---

# <img src="/ironcalc.png" class="tab-icon"> What is IronCalc?

IronCalc is a new, modern, work-in-progress spreadsheet engine and set of tools to work with spreadsheets in diverse settings.
<div class="glance">
  <div><span>Port</span><b><code>2080</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>2 services</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/ironcalc/IronCalc"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy IronCalc
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
