---
title: Dozzle
description: A guide to deploying Dozzle on TrueNAS Scale and via Docker Compose
published: true
date: 2025-06-08T18:39:50.611Z
tags: 
editor: markdown
dateCreated: 2024-11-04T21:31:46.439Z
---

![](/dozzle.png)

# What is Dozzle?

Dozzle is a small lightweight application with a web based interface to monitor Docker logs. It doesn’t store any log files. It is for live monitoring of your container logs only.

# Docker Compose

```yaml
services:
 dozzle:
   container_name: dozzle
   image: amir20/dozzle:latest
   ports:
     - '8888:8080'
   restart: unless-stopped
   volumes:
     - /var/run/docker.sock:/var/run/docker.sock
```

# YouTube Walkthrough

[https://youtu.be/LNDkGBOfv6Y](https://youtu.be/LNDkGBOfv6Y)