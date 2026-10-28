---
title: Warracker
description: A guide to deploying Warracker
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:09:22.115Z
---

# <img src="/warracker.png" class="tab-icon"> What is Warracker?
Warracker is an open source, self-hostable warranty tracker to monitor expirations, store receipts, files. You own the data, your rules!

<div class="glance">
  <div><span>Port</span><b><code>8005</code></b></div>
  <div><span>Deploy via</span><b>TrueNAS app or Docker compose</b></div>
  <div><span>Containers</span><b>2 services</b></div>
  <div><span>Depends on</span><b>Postgres</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/sassanix/warracker"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>


# 1 · Deploy Warracker
# tabs {.tabset}

## <img src="/truenas.png" class="tab-icon"> TrueNAS
1. Set a **Database Password**
1. Set the **Warracker Uploads Storage** and **Postgres Data Storage** to *Host Path*
1. Check the box for **Automatic Permissions** under the **Postgres Data Storage** section

## <img src="/docker.png" class="tab-icon"> Docker Compose
```yaml
services:
  warracker:
    image: ghcr.io/sassanix/warracker/main:latest
    ports:
      - "8005:80"
    volumes:
      - /mnt/tank/configs/warracker/uploads:/data/uploads
    env_file:
      - .env
    depends_on:
      warrackerdb:
        condition: service_healthy
    restart: unless-stopped

  warrackerdb:
    image: postgres:15-alpine
    volumes:
      - /mnt/tank/configs/warracker/db:/var/lib/postgresql/data
    env_file:
      - .env
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB"]
      interval: 5s
      timeout: 5s
      retries: 5
```

> Older versions of this guide used `/mnt/tank/configs/uploads` and `/mnt/tank/configs/db`, which can collide with other apps. If you already deployed with those paths, either keep them or move the folders into `/mnt/tank/configs/warracker/` before redeploying.
{.is-info}

### 1.1 env File

```env
# Warracker Environment Variables Configuration
# Copy this file to .env and customize the values for your deployment


### ** Database Configuration**

# Database connection settings
DB_HOST=warrackerdb
DB_NAME=warranty_db
DB_USER=warranty_user
DB_PASSWORD=warranty_password

# Database admin credentials (used for migrations and setup)
DB_ADMIN_USER=warracker_admin
DB_ADMIN_PASSWORD=change_this_password_in_production

# PostgreSQL-specific settings (for the database container)
POSTGRES_DB=warranty_db
POSTGRES_USER=warranty_user
POSTGRES_PASSWORD=warranty_password


###  Security Configuration**

# Application secret key for JWT tokens and Flask sessions
# Leave empty to auto-generate a random key on first start (it is persisted),
# or set your own with: openssl rand -hex 32
# Do NOT use a known default value here.
SECRET_KEY=

# JWT token expiration time (in hours)
JWT_EXPIRATION_HOURS=24


### Email/SMTP Configuration**

# SMTP server settings for sending notifications and password resets
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=youremail@gmail.com
SMTP_PASSWORD=your_email_password

# Optional SMTP settings
SMTP_USE_TLS=true
SMTP_USE_SSL=false
SMTP_SENDER_EMAIL=noreply@warracker.com


### **URL Configuration**

# Frontend URL (used for redirects and email links)
# IMPORTANT: Must match your public-facing URL for OIDC and email links to work
FRONTEND_URL=http://localhost:8005

# Application base URL (used for links in emails and redirects)
APP_BASE_URL=http://localhost:8005


### **File Upload Configuration**

# Maximum file upload size in megabytes
MAX_UPLOAD_MB=32

# Nginx maximum body size (should match or exceed MAX_UPLOAD_MB)
NGINX_MAX_BODY_SIZE_VALUE=32M


### **Performance & Memory Configuration**

# Memory optimization mode
# Options: optimized (default), ultra-light, performance
# - optimized: 2 workers, ~60-80MB RAM usage (recommended for most deployments)
# - ultra-light: 1 worker, ~40-50MB RAM usage (for very limited resources)
# - performance: 4 workers, ~150-200MB RAM usage (for high-traffic deployments)
WARRACKER_MEMORY_MODE=optimized


### **OIDC/SSO Configuration (Optional)**

# Enable/disable OIDC SSO functionality
OIDC_ENABLED=false

# OIDC Provider settings
# Provider name (affects button branding: google, github, microsoft, keycloak, etc.)
OIDC_PROVIDER_NAME=oidc

# OIDC client credentials (obtain from your OIDC provider)
OIDC_CLIENT_ID=
OIDC_CLIENT_SECRET=

# OIDC issuer URL (e.g., https://accounts.google.com)
OIDC_ISSUER_URL=

# OIDC scope (space-separated list of scopes)
OIDC_SCOPE=openid email profile

# OIDC admin group (optional, requires group scope)
OIDC_ADMIN_GROUP=

# OIDC-only mode: hide the username/password login form, OIDC login only
OIDC_ONLY_MODE=false

# Skip the login screen and redirect straight to the OIDC provider
# (only takes effect when OIDC_ONLY_MODE is also true)
OIDC_SKIP_LOGIN_SCREEN=false

### **Development/Debugging Configuration (Optional)**

# Flask environment (development/production)
FLASK_ENV=production

# Flask debug mode (true/false)
FLASK_DEBUG=false

# Flask run port (for development)
FLASK_RUN_PORT=5000

# Python unbuffered output (helpful for Docker logs)
PYTHONUNBUFFERED=1
```