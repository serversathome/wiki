---
title: Unpackerr
description: A guide to installing Unpackerr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-06-08T18:39:08.076Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:33:30.432Z
---

![](https://wiki.hydrology.cc/unpackerr-logo-text.png)

# What is Unpackerr?

Unpackerr runs as a daemon on your download host or seedbox. It checks for completed downloads and extracts them so Lidarr, Radarr, Readarr, and Sonarr may import them. If your problem is rar files getting stuck in your activity queue, then this is your solution.

Not a starr app user, and just need to extract files? We do that too. This application can run standalone and extract files found in a "watch" folder. In other words, you can configure this application to watch your download folder, and it will happily extract everything you download.

# Docker Compose

```yaml
services:

 unpackerr:
   image: golift/unpackerr
   container_name: unpackerr
   volumes:
     # You need at least this one volume mapped so Unpackerr can find your files to extract.
     # Make sure this matches your Starr apps; the folder mount (/downloads or /data) should be identical.
     - /mnt/tank/media:/media
   restart: unless-stopped
   # Get the user:group correct so unpackerr can read and write to your files.
   user: 568:568
   # What you see below are defaults for this compose. You only need to modify things specific to your environment.
   # Remove apps and feature configs you do not use or need.
   # ie. Remove all lines that begin with UN_CMDHOOK, UN_WEBHOOK, UN_FOLDER, UN_WEBSERVER, and other apps you do not use.
   environment:
     - TZ=America/New_York
     # General config
     - UN_DEBUG=false
     - UN_LOG_FILE=
     - UN_LOG_FILES=10
     - UN_LOG_FILE_MB=10
     - UN_INTERVAL=2m
     - UN_START_DELAY=1m
     - UN_RETRY_DELAY=5m
     - UN_MAX_RETRIES=3
     - UN_PARALLEL=1
     - UN_FILE_MODE=0644
     - UN_DIR_MODE=0755
     # Sonarr Config
     - UN_SONARR_0_URL=http://sonarr:8989
     - UN_SONARR_0_API_KEY=
     - UN_SONARR_0_PATHS_0=/media/downloads
     - UN_SONARR_0_PROTOCOLS=torrent
     - UN_SONARR_0_TIMEOUT=10s
     - UN_SONARR_0_DELETE_ORIG=false
     - UN_SONARR_0_DELETE_DELAY=5m
     # Radarr Config
     - UN_RADARR_0_URL=http://radarr:7878
     - UN_RADARR_0_API_KEY=
     - UN_RADARR_0_PATHS_0=/media/downloads
     - UN_RADARR_0_PROTOCOLS=torrent
     - UN_RADARR_0_TIMEOUT=10s
     - UN_RADARR_0_DELETE_ORIG=false
     - UN_RADARR_0_DELETE_DELAY=5m
     # Lidarr Config
     - UN_LIDARR_0_URL=http://lidarr:8686
     - UN_LIDARR_0_API_KEY=
     - UN_LIDARR_0_PATHS_0=/media/downloads
     - UN_LIDARR_0_PROTOCOLS=torrent
     - UN_LIDARR_0_TIMEOUT=10s
     - UN_LIDARR_0_DELETE_ORIG=false
     - UN_LIDARR_0_DELETE_DELAY=5m
     # Readarr Config
     - UN_READARR_0_URL=http://readarr:8787
     - UN_READARR_0_API_KEY=
     - UN_READARR_0_PATHS_0=/media/downloads
     - UN_READARR_0_PROTOCOLS=torrent
     - UN_READARR_0_TIMEOUT=10s
     - UN_READARR_0_DELETE_ORIG=false
     - UN_READARR_0_DELETE_DELAY=5m
     # Folder Config
     - UN_FOLDER_0_PATH=
     - UN_FOLDER_0_EXTRACT_PATH=
     - UN_FOLDER_0_DELETE_AFTER=10m
     - UN_FOLDER_0_DELETE_ORIGINAL=false
     - UN_FOLDER_0_DELETE_FILES=false
     - UN_FOLDER_0_MOVE_BACK=false
     # Webhook Config
     - UN_WEBHOOK_0_URL=
     - UN_WEBHOOK_0_NAME=
     - UN_WEBHOOK_0_NICKNAME=Unpackerr
     - UN_WEBHOOK_0_CHANNEL=
     - UN_WEBHOOK_0_TIMEOUT=10s
     - UN_WEBHOOK_0_SILENT=false
     - UN_WEBHOOK_0_IGNORE_SSL=false
     - UN_WEBHOOK_0_EXCLUDE_0=
     - UN_WEBHOOK_0_EVENTS_0=0
     - UN_WEBHOOK_0_TEMPLATE_PATH=
     - UN_WEBHOOK_0_CONTENT_TYPE=application/json
     # Command Hook Config
     - UN_CMDHOOK_0_COMMAND=
     - UN_CMDHOOK_0_NAME=
     - UN_CMDHOOK_0_TIMEOUT=10s
     - UN_CMDHOOK_0_SILENT=false
     - UN_CMDHOOK_0_SHELL=false
     - UN_CMDHOOK_0_EXCLUDE_0=
     - UN_CMDHOOK_0_EVENTS_0=0
     # Web Server Config
     - UN_WEBSERVER_METRICS=false
     - UN_WEBSERVER_LISTEN_ADDR=0.0.0.0:5656
     - UN_WEBSERVER_LOG_FILE=
     - UN_WEBSERVER_LOG_FILES=10
     - UN_WEBSERVER_LOG_FILE_MB=10
     - UN_WEBSERVER_SSL_CERT_FILE=
     - UN_WEBSERVER_SSL_KEY_FILE=
     - UN_WEBSERVER_URLBASE=/
     - UN_WEBSERVER_UPSTREAMS=
```

## Permissions & Folder Structure
- **PUID / PGID**: Ensure you use a user/group with the correct permissions for accessing media folders. TrueNAS SCALE defaults to 568:568 for apps.

- 📌 Refer to the [Folder-Structure](/Folder-Structure) guide for more details.

Obviously there is a lot here. Most of this can remain unmodified. For the apps you would like to link, go to that app's section and modify the URL, APIKEY, AND PATHS\_0. For example, for Sonarr I would need to enter the IP of the Sonarr server, the API Key, and enter /`media/downloads` for the path.

> For more info what all of these options do, as well as examples, [go here](https://unpackerr.zip/docs/install/configuration)
{.is-info}


# YouTube Walkthrough

For a video walkthrough of deploying this container:

[https://youtu.be/tXTHYMm6\_bE](https://youtu.be/tXTHYMm6_bE)