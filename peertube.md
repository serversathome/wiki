---
title: PeerTube
description: A guide to deploying PeerTube
published: true
date: 2025-11-21T11:44:57.476Z
tags: 
editor: markdown
dateCreated: 2025-11-21T11:44:57.476Z
---

# <img src="/peertube.png" class="tab-icon"> What is PeerTube?

PeerTube is a free, decentralized and federated video platform developed as an alternative to other platforms that centralize our data and attention, such as YouTube, Dailymotion or Vimeo.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy PeerTube
```yaml
services:
  peertube:
    image: chocobozzz/peertube:production-bookworm
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
    image: postgres:13-alpine
    env_file:
      - .env
    volumes:
      - /mnt/tank/configs/peertube/db:/var/lib/postgresql/data
    restart: unless-stopped
  redis:
    image: redis:6-alpine
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

## 1.1 env File
```yaml

```