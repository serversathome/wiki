---
title: BentoPDF
description: A guide to deploying BentoPDF via docker
published: true
date: 2026-01-30T13:06:42.872Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:22.226Z
---

# <img src="https://github.com/user-attachments/assets/f1a69d56-5850-4d83-95e8-052e25263fc8" class="tab-icon"> What is BentoPDF? 
BentoPDF is a powerful suite of tools that allow you to do all kinds of cool things with PDF files. Some examples are: siging PDF's, create or fill in forms, extract images and so much more.

> No datasets need to be created
{.is-info}

# <img src="/docker.png" class="tab-icon"> 1 · Deploy BentoPDF
```yaml
services:
  bentopdf:
    image: ghcr.io/alam00000/bentopdf:latest
    container_name: bentopdf
    ports:
      - 3060:8080
    restart: unless-stopped
```

> After deploying the container, simply go to `http://IP:3060` and start using the software.
{.is-success}
