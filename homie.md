---
title: Homie
description: A guide to deploying Homie
published: true
date: 2025-11-04T16:52:41.036Z
tags: 
editor: markdown
dateCreated: 2025-11-04T16:52:41.036Z
---

# 🏠 What is Homie?
A simple family utility app for managing household tasks with secure authentication.

**Features**: Shopping lists • Chores • Expiry tracking • Bills • Mobile-friendly

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Homie
```yaml
services:
  homie:
    image: ghcr.io/brramble/homie:latest
    container_name: homie
    restart: unless-stopped
    ports:
      - "5000:5000" 
    volumes:
      - /mnt/tank/configs/homie:/app/data
    environment:
      - SECRET_KEY=your-secret-key-here-change-in-production
      - FLASK_DEBUG=false
      - BASE_URL=http://localhost:5000
      - PORT=5000
      - DATABASE_PATH=/app/data/homie.db
      
      # ===== OIDC Authentication Configuration =====
      # Enable OIDC authentication (set to false to disable OIDC and prepare for local accounts)
      - OIDC_ENABLED=true
      # Replace with your OIDC provider details (required when OIDC is enabled)
      - OIDC_BASE_URL=https://your-oidc-provider.com
      - OIDC_CLIENT_ID=your-client-id
      - OIDC_CLIENT_SECRET=your-client-secret
      # Allowed email addresses (comma-separated list of who can access the app - used with OIDC)
      - ALLOWED_EMAILS=user1@example.com,user2@example.com,admin@example.com
      
      # ===== Local Users Configuration (when OIDC is disabled) =====
      # Local users list (simple names or full format username:email:Full Name)
      # - USERS=Emma,Cam,Alex
      # Enable secure cookies (set to true in production with HTTPS)
      - SESSION_COOKIE_SECURE=false
      - LOG_LEVEL=INFO
      - PYTHONUNBUFFERED=1
    
```