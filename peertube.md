---
title: PeerTube
description: A guide to deploying PeerTube
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:07:04.139Z
---

# <img src="/peertube.png" class="tab-icon"> What is PeerTube?

PeerTube is a free, decentralized and federated video platform developed as an alternative to other platforms that centralize our data and attention, such as YouTube, Dailymotion or Vimeo.

<div class="glance">
  <div><span>Port</span><b><code>9002</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>4 services</b></div>
  <div><span>Depends on</span><b>Postgres and Redis</b></div>
  <div><span>Difficulty</span><b class="difficulty intermediate">Intermediate</b></div>
</div>

# <img src="/docker.png" class="tab-icon"> 1 · Deploy PeerTube
```yaml
services:
  peertube:
    image: chocobozzz/peertube:production
    networks:
      default:
        ipv4_address: 172.18.0.42
        ipv6_address: fdab:e4b3:21a2:ef1b::42
    env_file:
      - .env
    ports:
      - 1935:1935
      - 9002:9000
    volumes:
      - /mnt/tank/configs/peertube/data:/data
      - /mnt/tank/configs/peertube/config:/config
    depends_on:
      - postgres
      - redis
      - postfix
    restart: unless-stopped
  postgres:
    image: postgres:17-alpine
    env_file:
      - .env
    volumes:
      - /mnt/tank/configs/peertube/db:/var/lib/postgresql/data
    restart: unless-stopped
  redis:
    image: redis:8-alpine
    volumes:
      - /mnt/tank/configs/peertube/redis:/data
    restart: unless-stopped
  postfix:
    image: mwader/postfix-relay
    env_file:
      - .env
    volumes:
      - /mnt/tank/configs/peertube/keys:/etc/opendkim/keys
    restart: unless-stopped
networks:
  default:
    enable_ipv6: true
    ipam:
      driver: default
      config:
        - subnet: 172.18.0.0/16
        - subnet: fdab:e4b3:21a2:ef1b::/64
```

> **Upgrading an existing install?** The old `production-bookworm` tag stopped at PeerTube 7.3.0 in December 2025; current images are Debian trixie based and tagged `production` (v8.x). Read the [PeerTube upgrade notes](https://docs.joinpeertube.org/install/docker) and the v8 changelog before switching. Postgres also moves from 13 (end of life) to 17, and a major Postgres version can't just be swapped: dump the database first (`docker compose exec postgres pg_dumpall -U <POSTGRES_USER> > peertube.sql`), start Postgres 17 with an empty `/mnt/tank/configs/peertube/db` folder, then restore the dump before starting PeerTube. Keep the old folder until you have confirmed it works. Also keep your existing `POSTGRES_USER` in the `.env` below: older versions of this page used `POSTGRES_USER=admin`, and switching it to `peertube` on an existing database means PeerTube logs in as a role that doesn't own its data.
{.is-warning}

## 1.1 env File
```env
# Database / Postgres service configuration
POSTGRES_USER=peertube
# Generate with: openssl rand -hex 16
POSTGRES_PASSWORD=<strong password>
POSTGRES_DB=peertube
PEERTUBE_DB_USERNAME=$POSTGRES_USER
PEERTUBE_DB_PASSWORD=$POSTGRES_PASSWORD
PEERTUBE_DB_SSL=false
PEERTUBE_DB_HOSTNAME=postgres

# PeerTube server configuration

PEERTUBE_WEBSERVER_HOSTNAME=peertube.example.com
#PEERTUBE_WEBSERVER_PORT=9002
#PEERTUBE_WEBSERVER_HTTPS=false
PEERTUBE_TRUST_PROXY=["127.0.0.1", "loopback", "172.18.0.0/16", "10.99.0.0/24"]

# Generate with: openssl rand -hex 32
PEERTUBE_SECRET=<your-generated-secret>

# E-mail configuration
# If you use a Custom SMTP server
#PEERTUBE_SMTP_USERNAME=
#PEERTUBE_SMTP_PASSWORD=
# Default to Postfix service name "postfix" in docker-compose.yml
# May be the hostname of your Custom SMTP server
PEERTUBE_SMTP_HOSTNAME=postfix
PEERTUBE_SMTP_PORT=25
PEERTUBE_SMTP_FROM=noreply@<MY DOMAIN>
PEERTUBE_SMTP_TLS=false
PEERTUBE_SMTP_DISABLE_STARTTLS=false
PEERTUBE_ADMIN_EMAIL=admin@admin.com

# Postfix service configuration
POSTFIX_myhostname=<MY DOMAIN>
# If you need to generate a list of sub/DOMAIN keys
# pass them as a whitespace separated string <DOMAIN>=<selector>
OPENDKIM_DOMAINS=<MY DOMAIN>=peertube
# see https://github.com/wader/postfix-relay/pull/18
OPENDKIM_RequireSafeKeys=no


# Comment these variables if your S3 provider does not support object ACL
PEERTUBE_OBJECT_STORAGE_UPLOAD_ACL_PUBLIC="public-read"
PEERTUBE_OBJECT_STORAGE_UPLOAD_ACL_PRIVATE="private"

#PEERTUBE_LOG_LEVEL=info
```

# 2 · Logging In
1. Navigate to the FQDN of your instance
1. Click the **Login** button in the top right
1. Search the logs for the auto generated first time password
1. The default user name is `root`