---
title: Authentik
description: A guide to deploying Authentik
published: true
date: 2025-09-23T11:29:20.546Z
tags: 
editor: markdown
dateCreated: 2025-09-23T11:23:01.113Z
---

# ![](/authentik.png){class="tab-icon"} What is Authentik?

Authentik is an IdP (Identity Provider) and SSO (Single Sign On) platform that is built with security at the forefront of every piece of code, every feature, with an emphasis on flexibility and versatility.

With authentik, site administrators, application developers, and security engineers have a dependable and secure solution for authentication in almost any type of environment. There are robust recovery actions available for the users and applications, including user profile and password management. You can quickly edit, deactivate, or even impersonate a user profile, and set a new password for new users or reset an existing password.

You can use authentik in an existing environment to add support for new protocols, so introducing authentik to your current tech stack doesn't present re-architecting challenges. We support all of the major providers, such as OAuth2, SAML, LDAP, and SCIM, so you can pick the protocol that you need for each application.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Authentik
```yaml
services:
  postgresql:
    env_file:
      - .env
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_USER: ${POSTGRES_USER} # Whatever user name you want
    healthcheck:
      interval: 30s
      retries: 5
      start_period: 20s
      test:
        - CMD-SHELL
        - pg_isready -d $${POSTGRES_DB} -U $${POSTGRES_USER}
      timeout: 5s
    image: docker.io/library/postgres:16-alpine
    restart: unless-stopped
    volumes:
      - database:/var/lib/postgresql/data # Leave this volume default
  
  redis:
    command: --save 60 1 --loglevel warning
    healthcheck:
      interval: 30s
      retries: 5
      start_period: 20s
      test:
        - CMD-SHELL
        - redis-cli ping | grep PONG
      timeout: 3s
    image: docker.io/library/redis:alpine
    restart: unless-stopped
    volumes:
      - redis:/data # Leave this volume default
  
  server:
    container_name: authentik
    command: server
    depends_on:
      postgresql:
        condition: service_healthy
      redis:
        condition: service_healthy
    env_file:
      - .env
    user: root
    environment:
      AUTHENTIK_POSTGRESQL__HOST: postgresql
      AUTHENTIK_POSTGRESQL__NAME: ${POSTGRES_DB}
      AUTHENTIK_POSTGRESQL__PASSWORD: ${POSTGRES_PASSWORD}
      AUTHENTIK_POSTGRESQL__USER: ${POSTGRES_USER}
      AUTHENTIK_REDIS__HOST: redis
      AUTHENTIK_SECRET_KEY: ${AUTHENTIK_SECRET_KEY}
    image: ghcr.io/goauthentik/server:2025.8.1
    ports:
      - 9000:9000 #http port
      - 9443:9443 #https port
    restart: unless-stopped
    volumes:
      - ${MNT}/media:/media # Point this to your authentik dataset, "media" folder will auto create on compose launch
      - ${MNT}/templates:/templates # Point this to your authentik dataset, "templates" folder will auto create on compose launch

  worker:
    container_name: authentik-worker
    command: worker
    depends_on:
      postgresql:
        condition: service_healthy
      redis:
        condition: service_healthy
    env_file:
      - .env
    environment:
      AUTHENTIK_POSTGRESQL__HOST: postgresql
      AUTHENTIK_POSTGRESQL__NAME: ${POSTGRES_DB}
      AUTHENTIK_POSTGRESQL__PASSWORD: ${POSTGRES_PASSWORD}
      AUTHENTIK_POSTGRESQL__USER: ${POSTGRES_USER}
      AUTHENTIK_REDIS__HOST: redis
      AUTHENTIK_SECRET_KEY: ${AUTHENTIK_SECRET_KEY}
    image: ghcr.io/goauthentik/server:2025.8.1
    restart: unless-stopped
    user: root
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock # Leave volume default
      - ${MNT}/media:/media # Point this to your authentik dataset, "media" folder will auto create on compose launch
      - ${MNT}/certs:/certs # Point this to your authentik dataset, "certs" folder will auto create on compose launch
      - ${MNT}/templates:/templates # Point this to your authentik dataset, "templates" folder will auto create on compose launch
```

1. Make a dataset in TrueNAS called `authentik` with 770 permmissions

4. In TrueNAS terminal copy and run this command `openssl rand -base64 36` the string that is output will replace `${POSTGRES_PASSWORD}` simply add it to your .env section (e.g. `${POSTGRES_PASSWORD}=36 character string`)
5. Next copy and run `openssl rand -base64 60` the string that is output will replace `${AUTHENTIK_SECRET_KEY}` simply add it to your .env section (e.g. `${AUTHENTIK_SECRET_KEY}=60 character string`)

