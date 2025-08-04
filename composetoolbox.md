---
title: Compose Toolbox
description: A guide to deploying Compose Toolbox via docker
published: true
date: 2025-08-04T13:11:29.269Z
tags: 
editor: markdown
dateCreated: 2025-08-04T13:11:29.269Z
---

# ![](/composetoolbox.png){class="tab-icon"} What is Compose Toolbox?

ComposeToolbox is a self-hostable web application that allows users to edit, validate, and get suggestions for your docker-compose.yml files. It has a fully featured code editor as well as a configuration panel that breaks down what exactly the compose file does.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Compose Toolbox

```yaml
services:
  composetoolbox:
    image: ghcr.io/bluegoosemedia/composetoolbox:latest
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    restart: unless-stopped
    container_name: composetoolbox

```

> Visit the [live site!](https://composetoolbox.com/)
{.is-success}
