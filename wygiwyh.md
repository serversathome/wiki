---
title: WYGIWYH
description: A guide to deploying WYGIWYH
published: true
date: 2025-12-01T21:30:39.326Z
tags: 
editor: markdown
dateCreated: 2025-12-01T21:18:41.991Z
---

# <img src="/wygiwyh.png" class="tab-icon"> What is WYGIWYH?
WYGIWYH (What You Get Is What You Have) is a powerful, principles-first finance tracker designed for people who prefer a no-budget, straightforward approach to managing their money. With features like multi-currency support, customizable transactions, and a built-in dollar-cost averaging tracker, WYGIWYH helps you take control of your finances with simplicity and flexibility.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy WYGIWYH
```yaml
services:
  web:
    image: eitchtee/wygiwyh:latest
    container_name: ${SERVER_NAME}
    ports:
      - "${OUTBOUND_PORT:-8000}:${INTERNAL_PORT:-8000}"
    env_file:
      - .env
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: postgres:15
    container_name: ${DB_NAME}
    restart: unless-stopped
    volumes:
      - /mnt/tank/configs/wygiwyh/db:/var/lib/postgresql/data/
    environment:
      - POSTGRES_USER=${SQL_USER}
      - POSTGRES_PASSWORD=${SQL_PASSWORD}
      - POSTGRES_DB=${SQL_DATABASE}
```

## env File
```yaml
SERVER_NAME=wygiwyh_server
DB_NAME=wygiwyh_pg

TZ=UTC # Change to your timezone. This only affects some async tasks.

DEBUG=false
URL = https://...
HTTPS_ENABLED=true
SECRET_KEY=dfmklnldb0rgjdkflj
DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]
OUTBOUND_PORT=9005

# Uncomment these variables to automatically create an admin account using these credentials on startup.
# After your first successfull login you can remove these variables from your file for safety reasons.
ADMIN_EMAIL=admin@admin.com
ADMIN_PASSWORD=admin

SQL_DATABASE=wygiwyh
SQL_USER=wygiwyh
SQL_PASSWORD=<INSERT A SAFE PASSWORD HERE>
SQL_HOST=${DB_NAME}
SQL_PORT=5432

# Gunicorn
WEB_CONCURRENCY=4

# App Configs
# Enable this if you want to keep deleted transactions in the database
ENABLE_SOFT_DELETE=false
# If ENABLE_SOFT_DELETE is true, transactions deleted for more than KEEP_DELETED_TRANSACTIONS_FOR days will be truly deleted. Set to 0 to keep all.
KEEP_DELETED_TRANSACTIONS_FOR=365

TASK_WORKERS=1 # This only work if you're using the single container option. Increase to have more open queues via procrastinate, you probably don't need to increase this.

# OIDC Configuration. Uncomment the lines below if you want to add OIDC login to your instance
#OIDC_CLIENT_NAME=""
#OIDC_CLIENT_ID=""
#OIDC_CLIENT_SECRET=""
#OIDC_SERVER_URL=""
#OIDC_ALLOW_SIGNUP=true
```