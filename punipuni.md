---
title: PuniPuni
description: A guide to deploying PuniPuni
published: true
date: 2026-01-15T15:31:06.375Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:40.075Z
---

# <img src="/punipuni.png" class="tab-icon"> What is PuniPuni?
PuniPuni is an user invitation system for Jellyfin, inspired by Wizarr. Built with Spring Boot and Angular, it allows user management for Jellyfin administrators through an interface and automation features.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy PuniPuni
```yaml
services:
  puni-app:
    image: kowlown/punipuni:latest
    ports:
      - "8089:8080"
    environment:
      - PUNI_APPLICATION_DATABASE_TYPE=postgresql
      - PUNI_APPLICATION_DATABASE_HOST=postgres
      - PUNI_APPLICATION_DATABASE_PORT=5432
      - PUNI_APPLICATION_DATABASE_USERNAME=dbuser
      - PUNI_APPLICATION_DATABASE_PASSWORD=dbpassword
      - PUNI_ADMIN_USERNAME=admin
      - PUNI_ADMIN_PASSWORD=adminpassword
      - PUNI_APPLICATION_WELCOME_PAGES=./welcome-pages
    depends_on:
      - postgres

  postgres:
    image: postgres:17
    environment:
      - POSTGRES_DB=database
      - POSTGRES_USER=dbuser
      - POSTGRES_PASSWORD=dbpassword
    volumes:
      - /mnt/tank/configs/punipuni/:/var/lib/postgresql/data
```

