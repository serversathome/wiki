---
title: Duplicati
description: A guide to deploying Duplicati on TrueNAS as well as via docker compose
published: true
date: 2025-07-28T15:38:35.676Z
tags: 
editor: markdown
dateCreated: 2025-07-28T10:28:55.007Z
---

# ![](/duplicati.png){class="tab-icon"} What is Duplicati?

Duplicati is an open-source backup client that securely stores encrypted, incremental, compressed remote backups of local files on cloud storage services and remote file servers. Duplicati supports not only various online backup services like OneDrive, Amazon S3, Backblaze, Rackspace Cloud Files, Tahoe LAFS, and Google Drive, but also any servers that support SSH/SFTP, WebDAV, or FTP. Duplicati uses standard components such as rdiff, zip, AESCrypt, and GnuPG.

# 1 · Deploy Duplicati
# {.tabset}

## <img src="/truenas.png" class="tab-icon"> TrueNAS
1. Set a **Password**
1. Set an **Encryption Key** to enable encryption (optional)
1. Change the **Data Storage** to *Host Path*
1. Add **Additional Storage**
a. Change the **Type** to *Host Path*
b. Set the **Mount Path** to `/storage`
c. Select a **Host Path**
1. Add **Additional Storage**
a. Change the **Type** to *Host Path*
b. Set the **Mount Path** to `/source`
c. Select a **Host Path**

## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml

services:
  duplicati:
    image: lscr.io/linuxserver/duplicati:latest
    container_name: duplicati
    environment:
      - PUID=0
      - PGID=0
      - TZ=America/New_York
      - SETTINGS_ENCRYPTION_KEY=changeme
    volumes:
      - /mnt/tank/configs/duplicati:/config
      - /mnt/tank/backups:/backups
      - /:/source
    ports:
      - 8200:8200
    restart: unless-stopped
```

# 2 · Duplicati Configuration

## 2.1 Adding a Backup
1. Click **+ Add Backup** in the left pane
1. Give it a **Name**
1. Optionally select **Encryption**
1. Click **Next**
1. Select **Storage Type**
1. Enter your **AuthID**
1. Click **Test Connection**
1. For **Source Data**, expand **Computer** and navigate to `/source` and select directories for backup
1. Click **Next**
1. Configure your **Schedule**
1. Click **Next**
1. Select a **Remote Volume Size**
1. Select a **Backup Retention** period
1. Click **Save**

## 2.2 Data Restoration
1. Click **Restore** in the left pane
1. Select files for restoration
1. Click **Continue**
1. Select destination of the files and file handling options
1. Check the box for **Restore read/write permissions**
1. Click **Restore**

## 2.3 Remote Management
1. Navigate to **Settings** in the left pane
1. Click the button to **Register for remote control**
1. Click the generated hyperlink to add Duplicati to your account
1. Click the blue **Register machine** button
1. Click the **Get Started** link in the left pane
1. Click **Next** to get to the **Connection key** step
1. Copy the conneciton key
a. Open the local Duplicati interface
b. Navigate to **Settings**
c. Scroll to the **Default Options** at the bottom
d. Click the **Edit as text** hyperlink
e. Paste the key
f. Click **OK**
1. Navigate back to the Duplicati web interface
1. Click **next Step** twice
1. Click **Back to Dashboard**


# <img src="/youtube.png" class="tab-icon"> 3 · Video