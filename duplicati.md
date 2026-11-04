---
title: Duplicati
description: A guide to deploying Duplicati on TrueNAS as well as via docker compose
published: true
date: '2026-11-04T16:23:22.042Z'
tags: []
editor: markdown
dateCreated: '2026-01-15T15:04:31.421Z'
---

# What is Duplicati?

Duplicati is an open-source backup client that securely stores encrypted, incremental, compressed remote backups of local files on cloud storage services and remote file servers. Duplicati supports not only various online backup services like OneDrive, Amazon S3, Backblaze, Rackspace Cloud Files, Tahoe LAFS, and Google Drive, but also any servers that support SSH/SFTP, WebDAV, or FTP. Duplicati uses standard components such as rdiff, zip, AESCrypt, and GnuPG.

# 1 · Deploy Duplicati
:::block-tabs
::block-tab{label="TrueNAS" icon="simple-icons:truenas"}
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

::

::block-tab{label="Docker Compose" icon="simple-icons:docker"}
```yaml

services:
  duplicati:
    image: lscr.io/linuxserver/duplicati:latest
    container_name: duplicati
    environment:
      - PUID=0
      - PGID=0
      - TZ=America/New_York
      - SETTINGS_ENCRYPTION_KEY=change-me # min 8 chars, alphanumeric. Generate with: openssl rand -hex 16
      - DUPLICATI__WEBSERVICE_PASSWORD=change-me # Web UI password; defaults to changeme if unset
    volumes:
      - /mnt/tank/configs/duplicati:/config
      - /mnt/tank/backups:/backups
      - /:/source
    ports:
      - 8200:8200
    restart: unless-stopped
```

::

:::

# 2 · Duplicati Configuration

## 2.1 Adding a Backup
1. Click **Add backup**, then choose **Add a new backup**
1. Under **Basic configuration**, give it a **Name** and optionally set an encryption passphrase
1. For **Source data**, expand **Computer** and navigate to `/source` and select directories for backup, then click **Continue**
1. For **Storage destination**, select the storage type and enter your credentials (e.g. **AuthID**), click **Test now**, then click **Continue**
1. Configure your **Schedule**, then click **Continue**
1. Under **Retention and miscellaneous**, select a **Remote Volume Size** and a **Backup Retention** period
1. Click **Submit**

> [!NOTE]
> In Duplicati versions before 2.4.0.1, the storage destination step comes before source data.

## 2.2 Data Restoration
1. Click **Restore** in the left pane
1. Select files for restoration
1. Click **Continue**
1. Select destination of the files and file handling options
1. Check the box for **Restore read/write permissions**
1. Click **Restore**

## 2.3 Remote Management
1. In the local Duplicati web interface, click **Click to register** in the top-right corner
1. Sign in to the Duplicati Console if prompted, and accept adding the machine to your account
1. The machine now shows up in the Duplicati Console, where you can manage it remotely


# :logos:youtube-icon: 3 · Video

::block-youtube{url="https://youtu.be/ZxSpwwtJ3aA"}
::
