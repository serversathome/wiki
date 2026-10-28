---
title: Backvault
description: A guide to deploying Backvault
published: true
date: 2026-10-28T12:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-01-15T15:03:18.042Z
---

# <img src="/screenshot_from_2025-12-04_11-37-02.png" class="tab-icon"> What is Backvault?
BackVault is a lightweight Dockerized multi-architecture service that periodically backs up your Bitwarden or Vaultwarden vaults into password-protected encrypted files. It’s designed for hands-free, secure, and automated backups using the official Bitwarden CLI.

<div class="glance">
  <div><span>Port</span><b><code>8080</code></b></div>
  <div><span>Deploy via</span><b>Docker compose</b></div>
  <div><span>Containers</span><b>1 service</b></div>
  <div><span>Difficulty</span><b class="difficulty beginner">Beginner</b></div>
  <div class="glance-links">
    <a href="https://github.com/mvfc/backvault"><i class="mdi mdi-github"></i>Project</a>
  </div>
</div>

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
      PUID: 568
      PGID: 568
    volumes:
      - /mnt/tank/configs/backvault/backups:/app/backups
      - /mnt/tank/configs/backvault/db:/app/db
    ports:
      - "8080:8080"
```

# 2 · Correcting Permissions
Backvault runs as the user set by `PUID` / `PGID` (568 above, the TrueNAS apps user), and your folders need to be owned by that same user:group. Assuming you used the file structure above run this command as root in the TrueNAS shell:
```bash
chown 568:568 /mnt/tank/configs/backvault -R
```

Now restart your container.

# 3 · View API Keys
In the Bitwarden or Vaultwarden web vault, navigate to **Settings → Security → Keys** and click **View API Key** to see your **Client ID** and **Client Secret**

# 4 · First-Run Setup
1. On first start, Backvault opens a one-time setup page. Browse to `http://your-server-ip:8080`
2. Enter your **master password**, the **Client ID** and **Client Secret** from step 3, and a **backup encryption password**
3. Submit the form. Backvault encrypts and stores these in `/app/db`, then the setup page shuts itself down and scheduled backups begin

> Keep the `/app/db` volume. As long as it persists you can restart or update the container without entering your credentials again. Don't lose the backup encryption password, you need it to open your backups.
{.is-info}

