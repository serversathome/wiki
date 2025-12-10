---
title: Warracker
description: A guide to deploying Warracker
published: true
date: 2025-12-10T19:03:48.898Z
tags: 
editor: markdown
dateCreated: 2025-12-10T18:58:22.402Z
---

# <img src="/warracker.png" class="tab-icon"> What is Warracker?
Warracker is an open source, self-hostable warranty tracker to monitor expirations, store receipts, files. You own the data, your rules!


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
      - /mnt/tank/configs/uploads:/data/uploads
    env_file:
      - .env
    depends_on:
      warrackerdb:
        condition: service_healthy
    restart: unless-stopped

  warrackerdb:
    image: postgres:15-alpine
    volumes:
      - /mnt/tank/configs/db:/var/lib/postgresql/data
    env_file:
      - .env
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $$POSTGRES_USER -d $$POSTGRES_DB"]
      interval: 5s
      timeout: 5s
      retries: 5
```

### 1.1 env File

```yaml
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
# IMPORTANT: Generate a strong, unique secret key for production!
SECRET_KEY=your_very_secret_flask_key_change_me

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