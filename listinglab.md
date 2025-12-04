---
title: Listing Lab
description: A guide to deploy Listing Lab
published: true
date: 2025-12-04T17:56:39.627Z
tags: 
editor: markdown
dateCreated: 2025-12-04T17:56:39.626Z
---

# <img src="/listing-lab.png" class="tab-icon"> What is Listing Lab?

A simple workspace for keeping track of homes you’re considering. Save listings, add notes, and share them with others. Listing Lab stays on top of changes, like price cuts and updates, so you always have an accurate view of the properties you're interested in. 

# <img src="/docker.png" class="tab-icon"> 1 · Listing Lab
```yaml
services:
  listing_lab:
    image: ghcr.io/adomi-io/listing-lab:latest
    ports:
      - "8069:8069"
      - "8072:8072"
    environment:
      ODOO_DB_HOST: ${ODOO_DB_HOST:-listing_lab_postgres}
      ODOO_DB_PORT: ${ODOO_DB_PORT:-5432}
      ODOO_DB_USER: ${ODOO_DB_USER:-listing_lab_user}
      ODOO_DB_PASSWORD: ${ODOO_DB_PASSWORD:-listing_lab_password}
      ODOO_DB_NAME: ${ODOO_DB_NAME:-listing_lab}
      ODOO_LOG_LEVEL: ${LOG_LEVEL:-info}
    restart: unless-stopped
    healthcheck:
      test: [ "CMD-SHELL", "curl -f http://localhost:8069/web/health || exit 1" ]
      start_period: 20s
      interval: 15s
      timeout: 15s
      retries: 5
    volumes:
      - /mnt/tank/configs/listinglab/odoo_data:/volumes/data

  listing_lab_postgres:
    image: postgres:16
    container_name: listing_lab_postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${ODOO_DB_USER:-listing_lab_user}
      POSTGRES_PASSWORD: ${ODOO_DB_PASSWORD:-listing_lab_password}
    ports:
      - "5432:5432"
    volumes:
      - /mnt/tank/configs/listinglab/postgres_data:/var/lib/postgresql/data

  rabbitmq:
    image: rabbitmq:3-management
    restart: unless-stopped
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: ${RABBITMQ_USER:-guest}
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASS:-guest}
    volumes:
      - /mnt/tank/configs/listinglab/rabbitmq_data:/var/lib/rabbitmq
    healthcheck:
      test: rabbitmq-diagnostics -q ping
      interval: 5s
      timeout: 30s
      retries: 3

  real_estate_scraper:
    image: ghcr.io/adomi-io/listing-lab-scraper:latest
    restart: unless-stopped
    environment:
      ODOO_URL: http://listing_lab:8069
      ODOO_DB_NAME: ${ODOO_DB_NAME:-listing_lab}
      ODOO_API_KEY: ${ODOO_API_KEY}
      RABBITMQ_HOST: rabbitmq
      RABBITMQ_PORT: 5672
      RABBITMQ_USER: ${RABBITMQ_USER:-guest}
      RABBITMQ_PASS: ${RABBITMQ_PASS:-guest}
      RABBITMQ_QUEUE: ${RABBITMQ_QUEUE:-property_scrape_queue}
      RABBITMQ_EXCHANGE: ${RABBITMQ_EXCHANGE:-property_exchange}
      RABBITMQ_ROUTING_KEY: ${RABBITMQ_ROUTING_KEY:-property.scrape}
    depends_on:
      listing_lab:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
```

# 2 · Setting Up the Scraper
1. 