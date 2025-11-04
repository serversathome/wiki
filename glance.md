---
title: Glance
description: A guide to deploying Glance dashboard
published: true
date: 2025-11-04T14:22:18.630Z
tags: 
editor: markdown
dateCreated: 2025-11-04T14:22:18.630Z
---

# <img src="/glance.png" class="tab-icon"> What is Glance?

A lightweight, highly customizable dashboard that displays your feeds in a beautiful, streamlined interface.
# <img src="/docker.png" class="tab-icon"> 1 · Deploy Glance
1. Navigate to your `stacks` directory
1. Run the following command **as root**:
    ```bash
    mkdir glance && cd glance && curl -sL https://github.com/glanceapp/docker-compose-template/archive/refs/heads/main.tar.gz | tar -xzf - --strip-components 2
    ```
1. Open/refresh your Dockge web UI
1. 