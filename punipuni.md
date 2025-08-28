---
title: PuniPuni
description: A guide to deploying PuniPuni
published: true
date: 2025-08-28T18:49:17.938Z
tags: 
editor: markdown
dateCreated: 2025-08-28T18:49:03.863Z
---

# <img src="/punipuni.png" class="tab-icon"> What is PuniPuni?
PuniPuni is an user invitation system for Jellyfin, inspired by Wizarr. Built with Spring Boot and Angular, it allows user management for Jellyfin administrators through an interface and automation features.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy PuniPuni
```yaml
version: '3.8'
services:
  puni-app:
    image: kowlown/punipuni:latest
    container_name: punipuni
    restart: unless-stopped
    ports:
      - "8089:8080"
    environment:
      - PUNI_ADMIN_USERNAME=admin
      - PUNI_ADMIN_PASSWORD=admin
    volumes:
      - /mnt/tank/configs/punipuni/:/welcome-pages
