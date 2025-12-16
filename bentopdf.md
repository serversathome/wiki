---
title: BentoPDF
description: A guide to deploying BentoPDF via docker
published: false
date: 2025-12-16
tags: 
editor: markdown
dateCreated: 2025-12-16
---

# <img width="263" height="512" alt="bentopdf" src="https://github.com/user-attachments/assets/f1a69d56-5850-4d83-95e8-052e25263fc8" /> {class="tab-icon"} What is BentoPDF? 
BentoPDF is a powerful suite of tools that allow you to do all kinds of cool things with PDF files. Some examples are: siging PDF's, create or fill in forms, extract images and so much more.

> No datasets need to be created
{.is-info}

# <img src="/docker.png" class="tab-icon"> 1 · Deploy BentoPDF
```yaml
services:
  bentopdf:
    image: bentopdf/bentopdf:latest
    container_name: bentopdf
    ports:
      - 3060:8080
    restart: unless-stopped
```

> After deploying the container, simply go to http://<IP>:3060 and start using the software.
{.is-success}
