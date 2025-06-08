---
title: Readarr/Calibre/Kavita
description: A guide to deploying Readarr/Calibre/Kavita using docker compose
published: true
date: 2025-06-08T18:40:00.890Z
tags: 
editor: markdown
dateCreated: 2025-01-04T20:18:26.385Z
---

> **This Page is Under Construction!** 
{.is-danger}


# Docker Compose

```yaml
services:
  readarr:
    image: lscr.io/linuxserver/readarr:develop
    container_name: readarr
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    volumes:
      - ./config:/config
      - /media:/media
    ports:
      - 8787:8787
    restart: unless-stopped
  calibre:
    image: lscr.io/linuxserver/calibre:latest
    container_name: calibre
    security_opt:
      - seccomp:unconfined #optional
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
      - PASSWORD= #optional
      - CLI_ARGS= #optional
    volumes:
      - ./config:/config
      - /media:/media
    ports:
      - 8092:8080
      - 8181:8181
      - 8093:8081
    restart: unless-stopped
  kavita:
    image: lscr.io/linuxserver/kavita:latest
    container_name: kavita
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    volumes:
      - ./config:/config
      - /media:/media
    ports:
      - 5000:5000
    restart: unless-stopped
```

Most likely you will need to change to PUID and PGID to a user and group which have the correct permissions to access the media folders you have set up. I use the 568 user/group because thats how TrueNAS is setup by default. See the [Folder Structure](https://wiki.serversatho.me/Folder-Structure) article for more info. 

The volumes have been modified for what I consider to be “common sense naming”. All containers have a directory or dataset under the `./configs` directory. See the [Folder Structure](https://wiki.serversatho.me/Folder-Structure) article for more info. 

# Readarr

## Root Folder

Navigate in Radarr to **Settings** > **Media Management** and at the bottom click the button to **Add Root Folder**. Select the `/media/books` directory.

![](/screenshot_from_2025-01-04_15-05-50.png)

## Download Client

Navigate in Radarr to **Settings** > **Download Client** and click the “+” icon. Click the box for **qBittorrent**. Change the host to the IP of the server and the port to the correct port which qbit is running on. Use the credentials you set up when you installed qBit. Leave all other options as default and click **Test**, then **Save** at the bottom.

# Calibre-Web

## Calibre Save to Disk template for Kavita

Calibre does not have sane defaults out of the box. You need to change the default settings to use your epubs in Kavita.

With the following save template you can have Calibre automatically create these for you, either by the set series name or the book title if no series is set. The filename will be the same, except with the series number added if the book is part of a series. In addition, it will convert any colons, used as sub-title, to a hyphen (assuming you leave a space after the colon).

1.  Open the Preferences in Calibre
2.  Click on ‘Saving books to Disk’ (found under Import/Export)
3.  Make sure ‘Save cover separately’ is **unchecked**.
4.  Make sure ‘Update metadata in saved copies’ is **checked**.
5.  Make sure ‘Save metadata in separate OPF file’ is **unchecked**.
6.  Adjust the ‘Save template’ to the following value;

`{series:'re(ifempty($,field('title')),':',' -')'}/{series:'re(ifempty($,field('title')),':',' -')'}{series_index:0>2s| - |}`

![](/calibre-saving-books-settings.30e6763f.png)

# Kavita

Add books folder.