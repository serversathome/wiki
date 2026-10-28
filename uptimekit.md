---
title: Uptime Kit
description: A guide to deploying Uptime Kit
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:09:15.349Z
---

# <img src="/uptimekit.png" class="tab-icon"> What is Uptime Kit?

A simple uptime monitoring dashboard to keep track of your websites and APIs. Get real-time updates, beautiful charts, and know exactly when something goes down.

<div class="glance">
  <div><span>Port</span><b><code>5173</code></b></div>
  <div><span>Deploy via</span><b>Docker compose (built from source)</b></div>
  <div><span>Containers</span><b>2 services</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/abhixdd/UptimeKit"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Uptime Kit
1. Navigate to your `stacks` directory in the TrueNAS shell
1. Run these commands as root:
    ```bash
    git clone https://github.com/abhixdd/UptimeKit
    mv UptimeKit uptimekit
    ```
1. Refresh your Dockge window
1. Start the stack
1. Open the dashboard at `http://YOUR-SERVER-IP:5173`

> If port 3000 is taken, change the backend's host port in the repo's compose.
{.is-warning}