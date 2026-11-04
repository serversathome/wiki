---
title: BentoPDF
description: A guide to deploying BentoPDF via docker
published: true
date: '2026-11-04T16:23:30.996Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:03:22.226Z'
---

# ![](https://github.com/user-attachments/assets/f1a69d56-5850-4d83-95e8-052e25263fc8 =28x) What is BentoPDF?
BentoPDF is a powerful suite of tools that allow you to do all kinds of cool things with PDF files. Some examples are: siging PDF's, create or fill in forms, extract images and so much more.

> [!NOTE]
> No datasets need to be created

# :logos:docker-icon: 1 · Deploy BentoPDF
```yaml
services:
  bentopdf:
    image: ghcr.io/alam00000/bentopdf-simple:latest
    container_name: bentopdf
    ports:
      - 3060:8080
    restart: unless-stopped
    # For IPv4-only hosts
    #environment:
    #  - DISABLE_IPV6=true
```

> [!NOTE]
> This uses the **Self-Hosted** build (`bentopdf-simple`), which upstream recommends for home and internal use. It has every PDF tool, just without the marketing pages. The `bentopdf` image without `-simple` is the commercial build that runs bentopdf.com.

> [!TIP]
> After deploying the container, simply go to `http://IP:3060` and start using the software.

# ![](https://github.com/user-attachments/assets/f1a69d56-5850-4d83-95e8-052e25263fc8 =28x) 2 · Documentation
<ol>
  <li><a href="https://www.bentopdf.com/">BentoPDF Website</a></li>
  <li><a href="https://www.github.com/alam00000/bentopdf">GitHub</a><br></li>
</ol>

