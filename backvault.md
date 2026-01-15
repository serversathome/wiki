---
title: Backvault
description: A guide to deploying Backvault
published: true
date: 2026-01-15T15:28:09.496Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:18.042Z
---

# <img src="/screenshot_from_2025-12-04_11-37-02.png" class="tab-icon"> What is Backvault?
BackVault is a lightweight Dockerized multi-architecture service that periodically backs up your Bitwarden or Vaultwarden vaults into password-protected encrypted files. It’s designed for hands-free, secure, and automated backups using the official Bitwarden CLI.

# <img src="/docker.png" class="tab-icon"> 1 · Deploy Backvault
```yaml
services:
  backvault:
    image: mvflc/backvault:latest
    container_name: backvault
    restart: unless-stopped
    environment:
      BW_SERVER: "https://vault.yourdomain.com"
      BACKUP_ENCRYPTION_MODE: "raw" # Use 'bitwarden' for the default format
      BACKUP_INTERVAL_HOURS: 12
      NODE_TLS_REJECT_UNAUTHORIZED: 0
      TZ: America/New_York
    volumes:
      - /mnt/tank/configs/backvault/backups:/app/backups
      - /mnt/tank/configs/backvault/db:/app/db
    ports:
      - "8080:8080"
```

# 2 · Correcting Permissions
Backvault needs your folders to be owned by user:group `1000`. Assuming you used the file structure above run this command as root in the TrueNAS shell:
```bash
chown 1000:1000 /mnt/tank/configs/backvault -R
```

Now restart your container.

# 3 · View API Keys
If you are using Vaultwarden, navigate to **Settings → Security → API Key** and click **View API Key** to see your **Client ID** and **Client Secret**

