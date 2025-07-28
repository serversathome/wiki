---
title: Duplicati
description: A guide to deploying Duplicati on TrueNAS as well as via docker compose
published: true
date: 2025-07-28T13:23:41.722Z
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
1. Set the **User ID** to `0` for access to all datasets
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
      - DUPLICATI_WEBSERVICE_PASSWORD=changeme
    volumes:
      - /mnt/tank/configs/duplicati:/config
      - /mnt/tank/backups:/backups
      - /:/source
    ports:
      - 8200:8200
    restart: unless-stopped
```

# 2 · Duplicati Configuration




# <img src="/youtube.png" class="tab-icon"> 3 · Video