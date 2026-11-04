---
title: Compose Toolbox
description: A guide to deploying Compose Toolbox via docker
published: true
date: '2026-11-04T16:24:04.434Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:03:54.032Z'
---

# What is Compose Toolbox?

ComposeToolbox is a self-hostable web application that allows users to edit, validate, and get suggestions for your docker-compose.yml files. It has a fully featured code editor as well as a configuration panel that breaks down what exactly the compose file does.

# :logos:docker-icon: 1 · Deploy Compose Toolbox

```yaml
services:
  composetoolbox:
    image: ghcr.io/bluegoosemedia/composetoolbox
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    restart: unless-stopped
    volumes:
      - ./composetoolbox/data:/app/data 

```

> [!TIP]
> Visit the [live site!](https://composetoolbox.com/)

