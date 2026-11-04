---
title: Uptime Kit
description: A guide to deploying Uptime Kit
published: true
date: '2026-11-04T16:23:47.968Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:09:15.349Z'
---

# What is Uptime Kit?

A simple uptime monitoring dashboard to keep track of your websites and APIs. Get real-time updates, beautiful charts, and know exactly when something goes down.

# :logos:docker-icon: 1 · Deploy Uptime Kit
1. Navigate to your `stacks` directory in the TrueNAS shell
1. Run these commands as root:
    ```bash
    git clone https://github.com/abhixdd/UptimeKit
    mv UptimeKit uptimekit
    ```
1. Refresh your Dockge window
1. Start the stack
1. Open the dashboard at `http://YOUR-SERVER-IP:5173`

> [!WARNING]
> If port 3000 is taken, change the backend's host port in the repo's compose.
